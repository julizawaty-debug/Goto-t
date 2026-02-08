<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Photo Location Finder</title>
    <script src="https://maps.googleapis.com/maps/api/js?key=YOUR_GOOGLE_MAPS_API_KEY_HERE"></script>
    <script src="https://cdn.jsdelivr.net/npm/exif-js@2.3.0/exif.min.js"></script>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; text-align: center; }
        #map { height: 400px; width: 100%; margin-top: 20px; border: 1px solid #ccc; }
        input { margin-bottom: 10px; }
        #location { margin-top: 10px; font-weight: bold; }
    </style>
</head>
<body>
    <h1>Upload a Photo to Find Its Location</h1>
    <p>Select an image file (e.g., from your phone's camera). This tool extracts GPS data from the photo's metadata and shows the location on a map.</p>
    <input type="file" id="fileInput" accept="image/*">
    <div id="map"></div>
    <p id="location">No location found yet.</p>

    <script>
        let map;

        document.getElementById('fileInput').addEventListener('change', function(event) {
            const file = event.target.files[0];
            if (file) {
                EXIF.getData(file, function() {
                    const lat = EXIF.getTag(this, 'GPSLatitude');
                    const lon = EXIF.getTag(this, 'GPSLongitude');
                    const latRef = EXIF.getTag(this, 'GPSLatitudeRef');
                    const lonRef = EXIF.getTag(this, 'GPSLongitudeRef');

                    if (lat && lon) {
                        // Convert EXIF GPS to decimal degrees
                        const latitude = convertGPS(lat, latRef);
                        const longitude = convertGPS(lon, lonRef);

                        // Update location text
                        document.getElementById('location').innerText = `Exact Location: Latitude ${latitude.toFixed(6)}, Longitude ${longitude.toFixed(6)}`;

                        // Initialize or update map
                        if (!map) {
                            map = new google.maps.Map(document.getElementById('map'), {
                                zoom: 15,
                                center: { lat: latitude, lng: longitude }
                            });
                        } else {
                            map.setCenter({ lat: latitude, lng: longitude });
                        }
                        new google.maps.Marker({
                            position: { lat: latitude, lng: longitude },
                            map: map
                        });
                    } else {
                        document.getElementById('location').innerText = 'No GPS data found in this photo. Try a photo taken with location services enabled.';
                        if (map) map.setCenter({ lat: 0, lng: 0 }); // Reset to equator if no data
                    }
                });
            }
        });

        function convertGPS(gpsArray, ref) {
            const degrees = gpsArray[0];
            const minutes = gpsArray[1];
            const seconds = gpsArray[2];
            let decimal = degrees + minutes / 60 + seconds / 3600;
            if (ref === 'S' || ref === 'W') decimal = -decimal;
            return decimal;
        }
    </script>
</body>
</html>
