# Exiftool

From https://www.reddit.com/r/googlephotos/comments/199qm59/i_want_to_upload_my_google_photo_takeout_data/
<code>https://exiftool.org/

exiftool -r -d %s -tagsfromfile "%d/%F.json" "-GPSAltitude<GeoDataAltitude" "-GPSLatitude<GeoDataLatitude" "-GPSLatitudeRef<GeoDataLatitude" "-GPSLongitude<GeoDataLongitude" "-GPSLongitudeRef<GeoDataLongitude" "-Keywords<Tags" "-Subject<Tags" "-Caption-Abstract<Description" "-ImageDescription<Description" "-DateTimeOriginal<PhotoTakenTimeTimestamp" -ext "*" -overwrite_original -progress --ext json C:\Photo</code>
