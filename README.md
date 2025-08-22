<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>台灣探險地圖 PWA</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
<style>
  #map { height: 100vh; width: 100%; }
</style>
</head>
<body>
<div id="map"></div>
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script>
  var map = L.map('map').setView([24.1477, 120.6736], 14);
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 19 }).addTo(map);

  // 讀取已探索區域和營地
  var explored = JSON.parse(localStorage.getItem('explored') || '[]');
  var camps = JSON.parse(localStorage.getItem('camps') || '[]');

  function addExploredCircle(lat, lng){
    var count = explored.filter(e => e.lat === lat && e.lng === lng).length;
    var colorOpacity = Math.min(0.1 + count*0.1, 0.8); // 探索次數越多顏色越深
    L.circle([lat, lng], {radius: 500, color:'green', fillColor:'green', fillOpacity: colorOpacity}).addTo(map);
  }

  explored.forEach(function(e){ addExploredCircle(e.lat, e.lng); });
  camps.forEach(function(c){
    var marker = L.marker([c.lat, c.lng]).addTo(map);
    marker.bindPopup('營地 (探索 '+c.count+' 次)');
  });

  // 點擊探索區域
  map.on('click', function(e){
    explored.push({lat:e.latlng.lat, lng:e.latlng.lng});
    localStorage.setItem('explored', JSON.stringify(explored));
    addExploredCircle(e.latlng.lat, e.latlng.lng);
  });

  // 長按設營地
  var pressTimer;
  map.on('mousedown', function(e){
    pressTimer = setTimeout(function(){
      var count = explored.filter(ex => ex.lat === e.latlng.lat && ex.lng === e.latlng.lng).length;
      var marker = L.marker(e.latlng).addTo(map);
      marker.bindPopup('營地 (探索 '+count+' 次)').openPopup();
      camps.push({lat:e.latlng.lat, lng:e.latlng.lng, count:count});
      localStorage.setItem('camps', JSON.stringify(camps));
    }, 800);
  });
  map.on('mouseup', function(e){ clearTimeout(pressTimer); });
</script>
</body>
</html>
