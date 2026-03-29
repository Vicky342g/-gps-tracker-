<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>📍 Live GPS Tracker - Real-time Location</title>
    <meta name="description" content="Real-time GPS location tracker with live map, speed, altitude and accuracy">
    <link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>📍</text></svg>">
    <style>
        /* Your existing CSS is perfect - keeping it exactly as is */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
        }

        .container {
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            padding: 30px;
            max-width: 800px;
            width: 100%;
        }

        h1 {
            text-align: center;
            color: #333;
            margin-bottom: 30px;
            font-size: 2.5em;
        }

        .controls {
            display: flex;
            gap: 15px;
            justify-content: center;
            margin-bottom: 30px;
            flex-wrap: wrap;
        }

        button {
            padding: 12px 24px;
            border: none;
            border-radius: 25px;
            background: #667eea;
            color: white;
            font-size: 16px;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        button:hover {
            background: #5a67d8;
            transform: translateY(-2px);
        }

        button:disabled {
            background: #ccc;
            cursor: not-allowed;
            transform: none;
        }

        .status {
            text-align: center;
            padding: 15px;
            border-radius: 10px;
            margin-bottom: 20px;
            font-weight: bold;
        }

        .status.success {
            background: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }

        .status.error {
            background: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }

        .status.info {
            background: #d1ecf1;
            color: #0c5460;
            border: 1px solid #bee5eb;
        }

        .location-info {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 10px;
            margin-bottom: 20px;
        }

        .coord-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 15px;
            margin-top: 15px;
        }

        .coord-item {
            background: white;
            padding: 15px;
            border-radius: 8px;
            text-align: center;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }

        .coord-label {
            font-size: 0.9em;
            color: #666;
            margin-bottom: 5px;
        }

        .coord-value {
            font-size: 1.3em;
            font-weight: bold;
            color: #333;
        }

        #map {
            height: 400px;
            width: 100%;
            border-radius: 10px;
            margin-top: 20px;
            background: #e9ecef;
        }

        .accuracy {
            text-align: center;
            margin-top: 10px;
            font-size: 0.9em;
            color: #666;
        }

        .extra-controls {
            display: flex;
            gap: 10px;
            justify-content: center;
            margin-top: 20px;
            flex-wrap: wrap;
        }

        .extra-controls button {
            padding: 8px 16px;
            font-size: 14px;
        }

        @media (max-width: 600px) {
            .controls {
                flex-direction: column;
                align-items: center;
            }
            
            button {
                width: 200px;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>📍 Live GPS Tracker</h1>
        
        <div class="controls">
            <button id="startTracking">🎯 Start Tracking</button>
            <button id="stopTracking" disabled>⏹️ Stop Tracking</button>
            <button id="getSingleLocation">📍 Current Location</button>
        </div>

        <div id="status" class="status info">👆 Click a button to start tracking your location</div>

        <div id="locationInfo" class="location-info" style="display: none;">
            <div class="coord-grid">
                <div class="coord-item">
                    <div class="coord-label">Latitude</div>
                    <div class="coord-value" id="latitude">-</div>
                </div>
                <div class="coord-item">
                    <div class="coord-label">Longitude</div>
                    <div class="coord-value" id="longitude">-</div>
                </div>
                <div class="coord-item">
                    <div class="coord-label">Altitude</div>
                    <div class="coord-value" id="altitude">-</div>
                </div>
                <div class="coord-item">
                    <div class="coord-label">Speed</div>
                    <div class="coord-value" id="speed">-</div>
                </div>
            </div>
            <div class="accuracy">
                📏 Accuracy: <span id="accuracy">-</span>m | 🕒 <span id="timestamp">-</span>
            </div>
        </div>

        <div id="map"></div>

        <div class="extra-controls" style="display: none;" id="extraControls">
            <button id="copyLocation">📋 Copy Location</button>
            <button id="openMaps">🗺️ Open in Maps</button>
            <button id="clearHistory">🗑️ Clear History</button>
            <button id="exportData">💾 Export Data</button>
        </div>
    </div>

    <script>
        class EnhancedLocationTracker {
            constructor() {
                this.watchId = null;
                this.trackHistory = [];
                this.isTracking = false;
                
                this.initializeElements();
                this.initializeMap();
                this.bindEvents();
            }

            initializeElements() {
                this.startBtn = document.getElementById('startTracking');
                this.stopBtn = document.getElementById('stopTracking');
                this.singleBtn = document.getElementById('getSingleLocation');
                this.statusDiv = document.getElementById('status');
                this.locationInfo = document.getElementById('locationInfo');
                this.extraControls = document.getElementById('extraControls');
                this.mapCanvas = document.getElementById('map');
                this.mapCtx = this.mapCanvas.getContext('2d');
            }

            initializeMap() {
                this.mapCanvas.width = this.mapCanvas.offsetWidth;
                this.mapCanvas.height = 400;
                this.drawBaseMap();
            }

            bindEvents() {
                this.startBtn.addEventListener('click', () => this.startTracking());
                this.stopBtn.addEventListener('click', () => this.stopTracking());
                this.singleBtn.addEventListener('click', () => this.getCurrentLocation());
                
                document.getElementById('copyLocation').addEventListener('click', () => this.copyLocation());
                document.getElementById('openMaps').addEventListener('click', () => this.openInMaps());
                document.getElementById('clearHistory').addEventListener('click', () => this.clearHistory());
                document.getElementById('exportData').addEventListener('click', () => this.exportData());
                
                window.addEventListener('resize', () => this.updateMapSize());
            }

            async checkGeolocationSupport() {
                if (!navigator.geolocation) {
                    this.showStatus('❌ Geolocation not supported by this browser', 'error');
                    return false;
                }
                return true;
            }

            async getCurrentLocation() {
                if (!await this.checkGeolocationSupport()) return;
                this.showStatus('🔍 Getting your current location...', 'info');
                
                navigator.geolocation.getCurrentPosition(
                    (position) => this.updateLocation(position, '✅ Location obtained!'),
                    (error) => this.handleLocationError(error),
                    { enableHighAccuracy: true, timeout: 10000, maximumAge: 60000 }
                );
            }

            startTracking() {
                if (!this.checkGeolocationSupport()) return;
                
                this.isTracking = true;
                this.showStatus('🚀 Starting live tracking...', 'info');
                this.updateButtons(true);
                this.extraControls.style.display = 'flex';

                this.watchId = navigator.geolocation.watchPosition(
                    (position) => {
                        this.updateLocation(position, '📍 Live tracking active');
                        this.updateMap(position);
                    },
                    (error) => this.handleLocationError(error),
                    { enableHighAccuracy: true, timeout: 5000, maximumAge: 0 }
                );
            }

            stopTracking() {
                if (this.watchId !== null) {
                    navigator.geolocation.clearWatch(this.watchId);
                    this.watchId = null;
                }
                this.isTracking = false;
                this.updateButtons(false);
                this.showStatus('⏹️ Tracking stopped', 'info');
            }

            updateButtons(tracking) {
                this.startBtn.disabled = tracking;
                this.stopBtn.disabled = !tracking;
                this.singleBtn.disabled = tracking;
            }

            updateLocation(position, message) {
                const coords = position.coords;
                const now = new Date();
                
                document.getElementById('latitude').textContent = coords.latitude.toFixed(6);
                document.getElementById('longitude').textContent = coords.longitude.toFixed(6);
                document.getElementById('altitude').textContent = coords.altitude ? 
                    coords.altitude.toFixed(0) + 'm' : 'N/A';
                document.getElementById('speed').textContent = coords.speed ? 
                    (coords.speed * 3.6).toFixed(1) + ' km/h' : '0 km/h';
                document.getElementById('accuracy').textContent = coords.accuracy.toFixed(0);
                document.getElementById('timestamp').textContent = now.toLocaleTimeString();

                this.locationInfo.style.display = 'block';
                this.showStatus(message, 'success');

                this.trackHistory.push({
                    timestamp: now.toISOString(),
                    lat: coords.latitude,
                    lng: coords.longitude,
                    alt: coords.altitude || 0,
                    speed: coords.speed || 0,
                    accuracy: coords.accuracy
                });
            }

            copyLocation() {
                const lat = document.getElementById('latitude').textContent;
                const lng = document.getElementById('longitude').textContent;
                navigator.clipboard.writeText(`${lat}, ${lng}`).then(() => {
                    this.showStatus('📋 Location copied to clipboard!', 'success');
                });
            }

            openInMaps() {
                const lat = document.getElementById('latitude').textContent;
                const lng = document.getElementById('longitude').textContent;
                window.open(`https://www.google.com/maps?q=${lat},${lng}`, '_blank');
            }

            clearHistory() {
                this.trackHistory = [];
                this.showStatus('🗑️ Tracking history cleared', 'info');
            }

            exportData() {
                const dataStr = JSON.stringify(this.trackHistory, null, 2);
                const dataBlob = new Blob([dataStr], {type: 'application/json'});
                const url = URL.createObjectURL(dataBlob);
                const link = document.createElement('a');
                link.href = url;
                link.download = `location-history-${new Date().toISOString().split('T')[0]}.json`;
                link.click();
            }

            handleLocationError(error) {
                let message = '';
                switch(error.code) {
                    case error.PERMISSION_DENIED:
                        message = "🚫 Location access denied. Enable permissions and refresh.";
                        break;
                    case error.POSITION_UNAVAILABLE:
                        message = "📡 Location unavailable. Check GPS signal.";
                        break;
                    case error.TIMEOUT:
                        message = "⏰ Location request timed out. Try again.";
                        break;
                    default:
                        message = "❓ Location error occurred.";
                        break;
                }
                this.showStatus(message, 'error');
                this.updateButtons(false);
            }

            showStatus(message, type) {
                this.statusDiv.textContent = message;
                this.statusDiv.className = `status ${type}`;
            }

            updateMapSize() {
                this.mapCanvas.width = this.mapCanvas.offsetWidth;
                this.drawBaseMap();
            }

            drawBaseMap() {
                const ctx = this.mapCtx;
                const width = this.mapCanvas.width;
                const height = this.mapCanvas.height;

                ctx.fillStyle = '#e9ecef';
                ctx.fillRect(0, 0, width, height);

                ctx.strokeStyle = '#dee2e6';
                ctx.lineWidth = 1;
                for (let i = 0; i < width; i += 50) {
                    ctx.beginPath();
                    ctx.moveTo(i, 0);
                    ctx.lineTo(i, height);
                    ctx.stroke();
                }
                for (let i = 0; i < height; i += 50) {
                    ctx.beginPath();
                    ctx.moveTo(0, i);
                    ctx.lineTo(width, i);
                    ctx.stroke();
                }
            }

            updateMap(position) {
                const ctx = this.mapCtx;
                const width = this.mapCanvas.width;
                const height = this.mapCanvas.height;

                this.drawBaseMap();

                const centerX = width / 2;
                const centerY = height / 2;
                const scale = 0.00008 * width;

                const x = centerX + (position.coords.longitude * scale);
                const y = centerY - (position.coords.latitude * scale);

                // Current position
                ctx.fillStyle = '#ff4757';
                ctx.shadowColor = 'rgba(255, 71, 87, 0.5)';
                ctx.shadowBlur = 15;
                ctx.beginPath();
                ctx.arc(x, y, 15, 0, 2 * Math.PI);
                ctx.fill();

                // Accuracy circle
                ctx.shadowBlur = 0;
                ctx.strokeStyle = 'rgba(255, 71, 87, 0.4)';
                ctx.lineWidth = 4;
                ctx.beginPath();
                ctx.arc(x, y, position.coords.accuracy * scale * 0.008, 0, 2 * Math.PI);
                ctx.stroke();

                // Label
                ctx.fillStyle = 'white';
                ctx.font = 'bold 14px Arial';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText('📍 YOU ARE HERE', x, y - 30);
                ctx.fillText(`${position.coords.accuracy.toFixed(0)}m`, x, y + 25);
            }
        }

        document.addEventListener('DOMContentLoaded', () => {
            new EnhancedLocationTracker();
        });
    </script>
</body>
</html>
// 1. Save locations to localStorage
localStorage.setItem('locations', JSON.stringify(this.trackHistory));

// 2. Share current location
const shareData = {
    title: 'My Location',
    text: `I'm at ${lat}, ${lng}`,
    url: `https://maps.google.com/?q=${lat},${lng}`
};
navigator.share(shareData);

// 3. Export track as GPX/KML
// 4. Integrate with real map APIs (Leaflet, Google Maps)
// 5. Add geofencing notifications[blackbox-output-code-454EKMDLEW.html](https://github.com/user-attachments/files/26332812/blackbox-output-code-454EKMDLEW.html)
