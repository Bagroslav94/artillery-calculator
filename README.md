
<!DOCTYPE html>
<html lang="cs">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dělostřelecká kalkulačka 81 mm EXPAL</title>
    <!-- Přidání Leaflet CSS a JS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background-color: #f4f4f4;
        }
        .calculator {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
            background-color: #fff;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }
        h1 {
            text-align: center;
            color: #333;
        }
        label {
            display: block;
            margin: 10px 0 5px;
            font-weight: bold;
        }
        select {
            width: 100%;
            padding: 8px;
            margin-bottom: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        button {
            padding: 10px;
            background-color: #28a745;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            margin: 5px;
        }
        button:hover {
            background-color: #218838;
        }
        .remove-btn {
            background-color: #dc3545;
        }
        .remove-btn:hover {
            background-color: #c82333;
        }
        .button-group {
            display: flex;
            justify-content: space-between;
            margin-bottom: 10px;
        }
        #map {
            height: 400px;
            margin-bottom: 20px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        #result {
            margin-top: 20px;
            padding: 10px;
            background-color: #e9ecef;
            border-radius: 4px;
            text-align: center;
        }
    </style>
</head>
<body>
    <div class="calculator">
        <h1>Kalkulačka pro minomet 81 mm EXPAL</h1>
        <div class="button-group">
            <button id="addGun">Přidat střelce</button>
            <button id="removeGun" class="remove-btn">Odebrat střelce</button>
            <button id="addTarget">Přidat cíl</button>
            <button id="removeTarget" class="remove-btn">Odebrat cíl</button>
        </div>
        <div id="map"></div>
        <form id="mortarForm">
            <label for="charge">Vyberte náplň:</label>
            <select id="charge" required>
                <option value="0 náplň">0 náplň (min. 100 m, max. 612 m)</option>
                <option value="1. náplň">1. náplň (min. 600 m, max. 1700 m)</option>
                <option value="2. náplň">2. náplň (min. 1000 m, max. 2850 m)</option>
                <option value="3. náplň">3. náplň (min. 1500 m, max. 4040 m)</option>
                <option value="4. náplň">4. náplň (min. 1900 m, max. 5170 m)</option>
                <option value="5. náplň">5. náplň (min. 2300 m, max. 6100 m)</option>
                <option value="6. náplň">6. náplň (min. 2600 m, max. 6590 m)</option>
            </select>

            <button type="submit">Vypočítat</button>
        </form>
        <div id="result"></div>
    </div>

    <script>
        // Tabulky náměrů pro 81 mm EXPAL, HE munice
        const firingTables = {
            "0 náplň": [
                { range: 100, elevation: 1515 },
                { range: 200, elevation: 1430 },
                { range: 300, elevation: 1340 },
                { range: 400, elevation: 1235 },
                { range: 500, elevation: 1115 },
                { range: 600, elevation: 900 },
                { range: 612, elevation: 800 },
                { maxRange: 612 }
            ],
            "1. náplň": [
                { range: 600, elevation: 1415 },
                { range: 700, elevation: 1385 },
                { range: 800, elevation: 1350 },
                { range: 900, elevation: 1315 },
                { range: 1000, elevation: 1280 },
                { range: 1100, elevation: 1240 },
                { range: 1200, elevation: 1200 },
                { range: 1300, elevation: 1155 },
                { range: 1400, elevation: 1105 },
                { range: 1500, elevation: 1050 },
                { range: 1600, elevation: 975 },
                { range: 1700, elevation: 800 },
                { maxRange: 1700 }
            ],
            "2. náplň": [
                { range: 1000, elevation: 1415 },
                { range: 1100, elevation: 1400 },
                { range: 1200, elevation: 1380 },
                { range: 1300, elevation: 1360 },
                { range: 1400, elevation: 1340 },
                { range: 1500, elevation: 1320 },
                { range: 1600, elevation: 1295 },
                { range: 1700, elevation: 1275 },
                { range: 1800, elevation: 1250 },
                { range: 1900, elevation: 1230 },
                { range: 2000, elevation: 1205 },
                { range: 2100, elevation: 1180 },
                { range: 2200, elevation: 1150 },
                { range: 2300, elevation: 1120 },
                { range: 2400, elevation: 1090 },
                { range: 2500, elevation: 1055 },
                { range: 2600, elevation: 1015 },
                { range: 2700, elevation: 970 },
                { range: 2800, elevation: 900 },
                { range: 2850, elevation: 800 },
                { maxRange: 2850 }
            ],
            "3. náplň": [
                { range: 1500, elevation: 1600 },
                { range: 1600, elevation: 1395 },
                { range: 1700, elevation: 1380 },
                { range: 1800, elevation: 1365 },
                { range: 1900, elevation: 1355 },
                { range: 2000, elevation: 1340 },
                { range: 2100, elevation: 1320 },
                { range: 2200, elevation: 1305 },
                { range: 2300, elevation: 1290 },
                { range: 2400, elevation: 1275 },
                { range: 2500, elevation: 1260 },
                { range: 2600, elevation: 1245 },
                { range: 2700, elevation: 1225 },
                { range: 2800, elevation: 1210 },
                { range: 2900, elevation: 1190 },
                { range: 3000, elevation: 1175 },
                { range: 3100, elevation: 1155 },
                { range: 3200, elevation: 1135 },
                { range: 3300, elevation: 1115 },
                { range: 3400, elevation: 1090 },
                { range: 3500, elevation: 1065 },
                { range: 3600, elevation: 1040 },
                { range: 3700, elevation: 1010 },
                { range: 3800, elevation: 975 },
                { range: 3900, elevation: 935 },
                { range: 4000, elevation: 870 },
                { range: 4040, elevation: 800 },
                { maxRange: 4040 }
            ],
            "4. náplň": [
                { range: 1900, elevation: 1410 },
                { range: 2000, elevation: 1400 },
                { range: 2100, elevation: 1390 },
                { range: 2200, elevation: 1380 },
                { range: 2300, elevation: 1365 },
                { range: 2400, elevation: 1355 },
                { range: 2500, elevation: 1340 },
                { range: 2600, elevation: 1330 },
                { range: 2700, elevation: 1320 },
                { range: 2800, elevation: 1310 },
                { range: 2900, elevation: 1295 },
                { range: 3000, elevation: 1285 },
                { range: 3100, elevation: 1278 },
                { range: 3200, elevation: 1260 },
                { range: 3300, elevation: 1245 },
                { range: 3400, elevation: 1235 },
                { range: 3500, elevation: 1220 },
                { range: 3600, elevation: 1205 },
                { range: 3700, elevation: 1190 },
                { range: 3800, elevation: 1175 },
                { range: 3900, elevation: 1160 },
                { range: 4000, elevation: 1145 },
                { range: 4100, elevation: 1130 },
                { range: 4200, elevation: 1110 },
                { range: 4300, elevation: 1095 },
                { range: 4400, elevation: 1070 },
                { range: 4500, elevation: 1050 },
                { range: 4600, elevation: 1030 },
                { range: 4700, elevation: 1010 },
                { range: 4800, elevation: 985 },
                { range: 4900, elevation: 955 },
                { range: 5000, elevation: 920 },
                { range: 5100, elevation: 865 },
                { range: 5170, elevation: 800 },
                { maxRange: 5170 }
            ],
            "5. náplň": [
                { range: 2300, elevation: 1410 },
                { range: 2400, elevation: 1400 },
                { range: 2500, elevation: 1390 },
                { range: 2600, elevation: 1380 },
                { range: 2700, elevation: 1370 },
                { range: 2800, elevation: 1360 },
                { range: 2900, elevation: 1350 },
                { range: 3000, elevation: 1345 },
                { range: 3100, elevation: 1335 },
                { range: 3200, elevation: 1325 },
                { range: 3300, elevation: 1315 },
                { range: 3400, elevation: 1305 },
                { range: 3500, elevation: 1295 },
                { range: 3600, elevation: 1280 },
                { range: 3700, elevation: 1270 },
                { range: 3800, elevation: 1260 },
                { range: 3900, elevation: 1250 },
                { range: 4000, elevation: 1240 },
                { range: 4100, elevation: 1230 },
                { range: 4200, elevation: 1215 },
                { range: 4300, elevation: 1205 },
                { range: 4400, elevation: 1195 },
                { range: 4500, elevation: 1180 },
                { range: 4600, elevation: 1165 },
                { range: 4700, elevation: 1155 },
                { range: 4800, elevation: 1140 },
                { range: 4900, elevation: 1125 },
                { range: 5000, elevation: 1110 },
                { range: 5100, elevation: 1095 },
                { range: 5200, elevation: 1080 },
                { range: 5300, elevation: 1065 },
                { range: 5400, elevation: 1045 },
                { range: 5500, elevation: 1025 },
                { range: 5600, elevation: 1005 },
                { range: 5700, elevation: 985 },
                { range: 5800, elevation: 960 },
                { range: 5900, elevation: 930 },
                { range: 6000, elevation: 890 },
                { range: 6100, elevation: 800 },
                { maxRange: 6100 }
            ],
            "6. náplň": [
                { range: 2600, elevation: 1410 },
                { range: 2700, elevation: 1400 },
                { range: 2800, elevation: 1390 },
                { range: 2900, elevation: 1385 },
                { range: 3000, elevation: 1375 },
                { range: 3100, elevation: 1365 },
                { range: 3200, elevation: 1360 },
                { range: 3300, elevation: 1350 },
                { range: 3400, elevation: 1340 },
                { range: 3500, elevation: 1335 },
                { range: 3600, elevation: 1325 },
                { range: 3700, elevation: 1315 },
                { range: 3800, elevation: 1310 },
                { range: 3900, elevation: 1300 },
                { range: 4000, elevation: 1290 },
                { range: 4100, elevation: 1280 },
                { range: 4200, elevation: 1270 },
                { range: 4300, elevation: 1260 },
                { range: 4400, elevation: 1250 },
                { range: 4500, elevation: 1240 },
                { range: 4600, elevation: 1230 },
                { range: 4700, elevation: 1220 },
                { range: 4800, elevation: 1210 },
                { range: 4900, elevation: 1200 },
                { range: 5000, elevation: 1190 },
                { range: 5100, elevation: 1180 },
                { range: 5200, elevation: 1165 },
                { range: 5300, elevation: 1155 },
                { range: 5400, elevation: 1140 },
                { range: 5500, elevation: 1130 },
                { range: 5600, elevation: 1120 },
                { range: 5700, elevation: 1105 },
                { range: 5800, elevation: 1090 },
                { range: 5900, elevation: 1080 },
                { range: 6000, elevation: 1065 },
                { range: 6100, elevation: 1045 },
                { range: 6200, elevation: 1030 },
                { range: 6300, elevation: 1015 },
                { range: 6400, elevation: 995 },
                { range: 6500, elevation: 975 },
                { range: 6590, elevation: 952 },
                { maxRange: 6590 }
            ]
        };

        // Inicializace mapy
        const map = L.map('map').setView([48.9012, 19.1234], 13); // Výchozí pozice (přibližně vaše souřadnice)

        // Satelitní dlaždice od Esri World Imagery (základní vrstva)
        const satelliteLayer = L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
            attribution: '© <a href="https://www.esri.com/">Esri</a>, i-cubed, USDA, USGS, AEX, GeoEye, Getmapping, Aerogrid, IGN, IGP, UPR-EGP, and the GIS User Community'
        }).addTo(map);

        // Překryvná vrstva s názvy měst a cest z OpenStreetMap (průhledná)
        const labelsLayer = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '© <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
            opacity: 0.5, // Nastavení průhlednosti, aby satelitní snímky zůstaly viditelné
            maxZoom: 19
        }).addTo(map);

        // Proměnné pro ukládání pozic střelce a cíle
        let gunMarker = null;
        let targetMarker = null;
        let gunLatLng = null;
        let targetLatLng = null;
        let addingGun = false;
        let addingTarget = false;

        // Funkce pro výpočet vzdálenosti (Haversinův vzorec)
        function calculateDistance(latLng1, latLng2) {
            const R = 6371000; // Poloměr Země v metrech
            const lat1 = latLng1.lat * Math.PI / 180;
            const lat2 = latLng2.lat * Math.PI / 180;
            const deltaLat = (latLng2.lat - latLng1.lat) * Math.PI / 180;
            const deltaLon = (latLng2.lng - latLng1.lng) * Math.PI / 180;

            const a = Math.sin(deltaLat / 2) * Math.sin(deltaLat / 2) +
                      Math.cos(lat1) * Math.cos(lat2) *
                      Math.sin(deltaLon / 2) * Math.sin(deltaLon / 2);
            const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
            return R * c; // Vzdálenost v metrech
        }

        // Funkce pro aktualizaci vzdálenosti
        function updateDistance() {
            if (gunLatLng && targetLatLng) {
                const distance = calculateDistance(gunLatLng, targetLatLng);
                document.getElementById('result').innerHTML = `Vzdálenost mezi střelcem a cílem: ${distance.toFixed(2)} metrů`;
            }
        }

        // Přidání střelce
        document.getElementById('addGun').addEventListener('click', function() {
            addingGun = true;
            addingTarget = false;
            document.getElementById('result').innerHTML = "Klikněte na mapu pro umístění střelce.";
        });

        // Odebrání střelce
        document.getElementById('removeGun').addEventListener('click', function() {
            if (gunMarker) {
                map.removeLayer(gunMarker);
                gunMarker = null;
                gunLatLng = null;
                document.getElementById('result').innerHTML = "Střelec byl odebrán.";
                if (targetLatLng) {
                    document.getElementById('result').innerHTML = "Cíl je stále označen. Přidejte střelce pro výpočet vzdálenosti.";
                }
            } else {
                document.getElementById('result').innerHTML = "Žádný střelec k odebrání.";
            }
        });

        // Přidání cíle
        document.getElementById('addTarget').addEventListener('click', function() {
            addingTarget = true;
            addingGun = false;
            document.getElementById('result').innerHTML = "Klikněte na mapu pro umístění cíle.";
        });

        // Odebrání cíle
        document.getElementById('removeTarget').addEventListener('click', function() {
            if (targetMarker) {
                map.removeLayer(targetMarker);
                targetMarker = null;
                targetLatLng = null;
                document.getElementById('result').innerHTML = "Cíl byl odebrán.";
                if (gunLatLng) {
                    document.getElementById('result').innerHTML = "Střelec je stále označen. Přidejte cíl pro výpočet vzdálenosti.";
                }
            } else {
                document.getElementById('result').innerHTML = "Žádný cíl k odebrání.";
            }
        });

        // Při kliknutí na mapu
        map.on('click', function(e) {
            const latlng = e.latlng;

            if (addingGun) {
                // Přidání střelce
                if (gunMarker) map.removeLayer(gunMarker); // Odebrání předchozí značky střelce
                gunLatLng = latlng;
                gunMarker = L.marker(latlng, { draggable: true }).addTo(map)
                    .bindPopup('Střelec')
                    .openPopup();

                // Umožnit přesun značky střelce
                gunMarker.on('dragend', function() {
                    gunLatLng = gunMarker.getLatLng();
                    updateDistance();
                });

                addingGun = false;
                document.getElementById('result').innerHTML = "Střelec byl přidán.";
                updateDistance();
            } else if (addingTarget) {
                // Přidání cíle
                if (targetMarker) map.removeLayer(targetMarker); // Odebrání předchozí značky cíle
                targetLatLng = latlng;
                targetMarker = L.marker(latlng, { draggable: true }).addTo(map)
                    .bindPopup('Cíl')
                    .openPopup();

                // Umožnit přesun značky cíle
                targetMarker.on('dragend', function() {
                    targetLatLng = targetMarker.getLatLng();
                    updateDistance();
                });

                addingTarget = false;
                document.getElementById('result').innerHTML = "Cíl byl přidán.";
                updateDistance();
            }
        });

        // Funkce pro získání náměru z tabulky
        function getElevation(distance, charge) {
            const table = firingTables[charge];
            const maxRange = table.maxRange;
            const minRange = table[0].range; // Minimální vzdálenost je první hodnota v tabulce

            // Kontrola dosahu
            if (distance < minRange) {
                return { elevation: null, error: `Vzdálenost je menší než minimální dosah zvolené náplně (${minRange} m)!` };
            }
            if (distance > maxRange) {
                return { elevation: null, error: `Vzdálenost přesahuje maximální dosah zvolené náplně (${maxRange} m)!` };
            }

            // Najdeme nejbližší hodnoty pro interpolaci
            let lower = null;
            let upper = null;
            for (const entry of table) {
                if (entry.range) {
                    if (entry.range <= distance && (!lower || entry.range > lower.range)) {
                        lower = entry;
                    }
                    if (entry.range >= distance && (!upper || entry.range < upper.range)) {
                        upper = entry;
                    }
                }
            }

            // Pokud je vzdálenost přesně v tabulce
            if (lower.range === distance) {
                return { elevation: lower.elevation };
            }

            // Lineární interpolace
            const fraction = (distance - lower.range) / (upper.range - lower.range);
            const elevation = lower.elevation + fraction * (upper.elevation - lower.elevation);

            return { elevation: elevation };
        }

        document.getElementById('mortarForm').addEventListener('submit', function(e) {
            e.preventDefault();

            // Kontrola, zda jsou vybrány obě pozice
            if (!gunLatLng || !targetLatLng) {
                document.getElementById('result').innerHTML = "Prosím, označte na mapě pozici střelce a cíle!";
                return;
            }

            // Získání hodnot z formuláře
            const charge = document.getElementById('charge').value;

            try {
                // Výpočet vzdálenosti (Haversinův vzorec)
                const distance = calculateDistance(gunLatLng, targetLatLng);

                // Výpočet azimutu
                const lat1 = gunLatLng.lat * Math.PI / 180;
                const lat2 = targetLatLng.lat * Math.PI / 180;
                const deltaLon = (targetLatLng.lng - gunLatLng.lng) * Math.PI / 180;

                const y = Math.sin(deltaLon) * Math.cos(lat2);
                const x = Math.cos(lat1) * Math.sin(lat2) -
                          Math.sin(lat1) * Math.cos(lat2) * Math.cos(deltaLon);
                let azimuth = Math.atan2(y, x) * 180 / Math.PI;
                if (azimuth < 0) azimuth += 360; // Normalizace na 0–360 stupňů
                const azimuthMils = azimuth * 17.7778; // Převod na MILS

                // Získání náměru z tabulky
                const elevationData = getElevation(distance, charge);
                if (elevationData.error) {
                    document.getElementById('result').innerHTML = elevationData.error;
                    return;
                }

                const elevationMils = elevationData.elevation;

                // Výsledek
                document.getElementById('result').innerHTML = `
                    <strong>Výsledek (munice: HE):</strong><br>
                    Střelec (WGS84): ${gunLatLng.lat.toFixed(6)}, ${gunLatLng.lng.toFixed(6)}<br>
                    Cíl (WGS84): ${targetLatLng.lat.toFixed(6)}, ${targetLatLng.lng.toFixed(6)}<br>
                    Vzdálenost: ${distance.toFixed(2)} metrů<br>
                    Náměr (elevace): ${elevationMils.toFixed(2)} MILS<br>
                    Azimut: ${azimuthMils.toFixed(2)} MILS<br>
                    Náplň: ${charge}<br>
                    Poznámka: Hodnoty jsou z tabulek pro HE munici. Zohledněte meteorologické podmínky.
                `;
            } catch (error) {
                document.getElementById('result').innerHTML = "Chyba při zpracování souřadnic! Zkontrolujte označení na mapě.";
            }
        });
    </script>
</body>
</html>

