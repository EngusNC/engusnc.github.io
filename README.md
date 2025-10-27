<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sélecteur de localisation</title>
    <script src='https://api.mapbox.com/mapbox-gl-js/v3.0.1/mapbox-gl.js'></script>
    <link href='https://api.mapbox.com/mapbox-gl-js/v3.0.1/mapbox-gl.css' rel='stylesheet' />
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, sans-serif;
            background: #f5f5f5;
        }
        #map {
            width: 100%;
            height: 500px;
        }
        .info-panel {
            background: white;
            padding: 20px;
            border-radius: 8px;
            margin: 20px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        }
        .info-panel h2 {
            margin-bottom: 15px;
            color: #333;
            font-size: 20px;
        }
        .location-info {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 6px;
            margin-top: 15px;
        }
        .location-info p {
            margin: 8px 0;
            color: #555;
            line-height: 1.6;
        }
        .location-info strong {
            color: #333;
            font-weight: 600;
        }
        .instruction {
            background: #e3f2fd;
            padding: 12px;
            border-radius: 6px;
            margin-bottom: 15px;
            color: #1565c0;
            border-left: 4px solid #2196f3;
        }
        .copy-btn {
            background: #2196f3;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 14px;
            margin-top: 10px;
            margin-right: 10px;
            transition: background 0.3s;
        }
        .copy-btn:hover {
            background: #1976d2;
        }
        .copy-btn:active {
            background: #0d47a1;
        }
        .status {
            padding: 10px;
            margin: 20px;
            border-radius: 6px;
            background: #fff3cd;
            color: #856404;
            text-align: center;
        }
    </style>
</head>
<body>
    <div id="status" class="status">Chargement de la carte...</div>
    <div id="map"></div>

    <div class="info-panel">
        <h2>📍 Localisation sélectionnée</h2>
        <div class="instruction">
            Cliquez n'importe où sur la carte pour sélectionner une localisation
        </div>
        <div class="location-info" id="locationInfo">
            <p>Aucune localisation sélectionnée. Cliquez sur la carte pour commencer.</p>
        </div>
    </div>

    <script>
        // Attendre que le DOM soit complètement chargé
        document.addEventListener('DOMContentLoaded', function() {
            const statusDiv = document.getElementById('status');
            
            // Vérifier que mapboxgl est disponible
            if (typeof mapboxgl === 'undefined') {
                statusDiv.textContent = '❌ Erreur: Mapbox GL JS n\'a pas pu se charger';
                statusDiv.style.background = '#ffebee';
                statusDiv.style.color = '#c62828';
                return;
            }

            statusDiv.textContent = '✅ Mapbox chargé, initialisation...';
            
            // Clé API Mapbox
            mapboxgl.accessToken = 'pk.eyJ1IjoiZW5ndXNuYyIsImEiOiJjbWgxa3pidWswOW13MmtzZnlldzVuYWc3In0.FpHT1EXM_T5dZ4fO-VDMrA';

            // Données de localisation
            let locationData = {
                latitude: null,
                longitude: null,
                address: null
            };

            let marker = null;

            // Initialiser la carte
            const map = new mapboxgl.Map({
                container: 'map',
                style: 'mapbox://styles/mapbox/streets-v12',
                center: [1.4442, 43.6047],
                zoom: 12
            });

            // Ajouter les contrôles
            map.addControl(new mapboxgl.NavigationControl());
            map.addControl(new mapboxgl.GeolocateControl({
                positionOptions: { enableHighAccuracy: true },
                trackUserLocation: true
            }));

            // Quand la carte est chargée
            map.on('load', function() {
                statusDiv.style.display = 'none';
                console.log('Carte chargée avec succès');
            });

            // Gérer les erreurs
            map.on('error', function(e) {
                statusDiv.textContent = '❌ Erreur: ' + e.error.message;
                statusDiv.style.background = '#ffebee';
                statusDiv.style.color = '#c62828';
                statusDiv.style.display = 'block';
            });

            // Fonction pour obtenir l'adresse
            async function getAddress(lng, lat) {
                try {
                    const response = await fetch(
                        `https://api.mapbox.com/geocoding/v5/mapbox.places/${lng},${lat}.json?access_token=${mapboxgl.accessToken}&language=fr`
                    );
                    const data = await response.json();
                    
                    if (data.features && data.features.length > 0) {
                        return data.features[0].place_name;
                    }
                    return 'Adresse non trouvée';
                } catch (error) {
                    console.error('Erreur géocodage:', error);
                    return 'Erreur de géocodage';
                }
            }

            // Mettre à jour l'affichage
            function updateLocationDisplay() {
                const infoDiv = document.getElementById('locationInfo');
                
                if (locationData.latitude && locationData.longitude) {
                    infoDiv.innerHTML = `
                        <p><strong>Latitude :</strong> ${locationData.latitude}</p>
                        <p><strong>Longitude :</strong> ${locationData.longitude}</p>
                        <p><strong>Adresse :</strong> ${locationData.address || 'Chargement...'}</p>
                        <button class="copy-btn" onclick="copyCoordinates()">📋 Coordonnées</button>
                        <button class="copy-btn" onclick="copyAddress()">📋 Adresse</button>
                        <button class="copy-btn" onclick="copyAll()">📋 Tout</button>
                    `;
                }
            }

            // Fonctions de copie
            window.copyCoordinates = function() {
                const text = `${locationData.latitude}, ${locationData.longitude}`;
                navigator.clipboard.writeText(text).then(() => alert('Coordonnées copiées ! 📋'));
            };

            window.copyAddress = function() {
                navigator.clipboard.writeText(locationData.address).then(() => alert('Adresse copiée ! 📋'));
            };

            window.copyAll = function() {
                const text = `Latitude: ${locationData.latitude}\nLongitude: ${locationData.longitude}\nAdresse: ${locationData.address}`;
                navigator.clipboard.writeText(text).then(() => alert('Tout copié ! 📋'));
            };

            // Clic sur la carte
            map.on('click', async function(e) {
                const { lng, lat } = e.lngLat;
                
                locationData.latitude = lat.toFixed(6);
                locationData.longitude = lng.toFixed(6);
                locationData.address = 'Chargement...';
                
                if (marker) marker.remove();
                
                marker = new mapboxgl.Marker({ color: '#FF0000' })
                    .setLngLat([lng, lat])
                    .addTo(map);
                
                updateLocationDisplay();
                
                const address = await getAddress(lng, lat);
                locationData.address = address;
                updateLocationDisplay();
                
                // Envoyer au parent (Tally)
                if (window.parent) {
                    window.parent.postMessage({
                        type: 'location-selected',
                        data: locationData
                    }, '*');
                }
                
                console.log('Localisation:', locationData);
            });
        });
    </script>
</body>
</html>
