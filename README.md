# Encoding OS Terrain 50 ASCII Grid data as Terrain-RGB tiles

### Introduction

OS Terrain 50 is a digital terrain model of the landscape, designed for landscape visualisation and analysis over large
areas. The ASCII Grid data (heighted points with regular 50 metre post spacing) can be downloaded [here](https://www.ordnancesurvey.co.uk/business-and-government/products/terrain-50.html).

<br>

### Requirements

1) Download/Install GDAL<br>https://www.gdal.org/
2) Install rio-rgbify<br>https://github.com/mapbox/rio-rgbify
3) Download OS Open Zoomstack GeoPackage<br>https://www.ordnancesurvey.co.uk/business-and-government/products/os-open-zoomstack.html

<br>

### Processing

```sh
# -- Extract ASCII grid files to output directory
find ~/Documents/Terrain50/zip -name *.zip -exec unzip {} "*.asc" -d ~/Desktop/Terrain50/asc \;

# -- Build VRT (Virtual Dataset) for ASCII grid files
find ~/Documents/Terrain50/asc -name "*.asc" | sort > terrain50_filelist.txt
gdalbuildvrt -vrtnodata "-9999" -input_file_list terrain50_filelist.txt terrain50.vrt

# -- Reproject VRT to Web Mercator and output to GeoTIFF
gdalwarp -overwrite -s_srs EPSG:27700 -t_srs EPSG:3857 terrain50.vrt _terrain50.tif

# -- Extract land polygons from OS Open Zoomstack GeoPackage; and use as a mask/cutline to clip DTM
# -- This overcomes an issue where the tile edges are visible in the hillshade layer
ogr2ogr -f "ESRI Shapefile" land.shp OSOpen_Zoomstack_GPKG.gpkg -t_srs EPSG:3857 -sql "SELECT * FROM land"
gdalwarp -cutline land.shp -t_srs EPSG:3857 _terrain50.tif terrain50.tif

# -- List information about the raster dataset
# -- Use [-mm] to force computation of the actual min/max values for each band 
gdalinfo -mm terrain50.tif

# -- Encode raster to Terrain-RGB mbtiles
# -- The base value [-b] should be equal to the computed min value returned from the gdalinfo command
rio rgbify -b -119.8 -i 0.05 --max-z 9 --min-z 5 --format png terrain50.tif terrain50.mbtiles
```

<br>

[MBUtil](https://github.com/mapbox/mbutil) can subsequently be used to export the MBTiles output to a directory of (PNG) files.
