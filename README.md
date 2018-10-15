# Encoding OS Terrain 50 ASCII Grid data as Terrain-RGB tiles

### Introduction
OS Terrain 50 is a digital terrain model of the landscape, designed for landscape visualisation and analysis over large
areas. The ASCII Grid data (heighted points with regular 50 metre post spacing) can be downloaded [here](https://www.ordnancesurvey.co.uk/business-and-government/products/terrain-50.html).

### Prerequisites
1) Download/Install GDAL<br>https://www.gdal.org/
2) Install rio-rgbify<br>https://github.com/mapbox/rio-rgbify

### Processing
```sh
# -- Extract ASCII grid files to output directory
find ~/Documents/Terrain50/zip -name "*.zip" -exec unzip {} "*.asc" -d ~/Documents/Terrain50/asc \;

# -- Build VRT (Virtual Dataset) for ASCII grid files
find ~/Documents/Terrain50/asc -name "*.asc" | sort > terrain50_filelist.txt
gdalbuildvrt -vrtnodata -9999 -input_file_list terrain50_filelist.txt terrain50.vrt

# -- Reproject VRT to Web Mercator and output as GeoTIFF
gdalwarp -s_srs EPSG:27700 -t_srs EPSG:3857 -srcnodata -9999 -dstnodata 0 -co COMPRESS=DEFLATE terrain50.vrt terrain50.tif

# -- Encode raster to Terrain-RGB mbtiles
rio rgbify -b -10000 -i 0.1 --max-z 9 --min-z 5 --format png terrain50.tif terrain50.mbtiles
```

[MBUtil](https://github.com/mapbox/mbutil) can subsequently be used to export the MBTiles output to a directory of (PNG) files.

<br>

## Uploading the generated output to Mapbox Studio

The MBTiles format must be used if there is a requirement to upload the files to Mapbox.<br>
Additionally the same encoding as Terrain-RGB must be used (with a base value of `-10000` and an interval of `0.1`).<br>
See https://github.com/mapbox/rio-rgbify/issues/19 for more detailed information.
