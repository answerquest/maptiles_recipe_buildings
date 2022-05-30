# maptiles_recipe_buildings
recipes to serve vector and raster map tiles of datasets by https://github.com/microsoft/GlobalMLBuildingFootprints

## Latest status: 
- as of: 2022-05-30
- Vector AND Raster map tiles layers working, see: https://server.nikhilvj.co.in/buildings1/ (Tileserver GL deployment)
- Vector tile url: `https://server.nikhilvj.co.in/buildings1/data/buildings-z13/{z}/{x}/{y}.pbf`
- Raster webp url: `https://server.nikhilvj.co.in/buildings1/styles/basic/{z}/{x}/{y}.webp` (reccommended, is faster than png)
- Raster png url: `https://server.nikhilvj.co.in/buildings1/styles/basic/{z}/{x}/{y}.png`


## Converting to mbtiles using Tippecanoe
- https://github.com/mapbox/tippecanoe 
- Command: (ran as a background task on a VPS server)  
```
nohup tippecanoe -o india_buildings_z14.mbtiles \
-n india_buildings_z14 \
-N "from https://github.com/microsoft/GlobalMLBuildingFootprints India download, processed by Nikhil VJ, https://nikhilvj.co.in" \
-z14 --extend-zooms-if-still-dropping \
--drop-densest-as-needed --drop-smallest-as-needed \
--coalesce-smallest-as-needed \
--coalesce-densest-as-needed \
--read-parallel --exclude-all \
India.geojsonl &
```
- Created a 2.5GB .mbtiles file in about 3 hrs. Did not consume too much RAM as it didn't have to load the entire data into memory.


## Serving vector tiles
- Using TileServer GL: https://github.com/maptiler/tileserver-gl/ , https://tileserver.readthedocs.io/
- loading the generated .mbtiles straight into tileserver-gl without any config is working:
```
docker run --rm -it -v $(pwd):/data -p 7100:8080 -p 7101:80 maptiler/tileserver-gl --verbose -b 0.0.0.0 -u "https://server.nikhilvj.co.in/buildings1/"
```
- output:
```
Starting tileserver-gl v3.1.1
No MBTiles specified, using india_buildings_z14.mbtiles
[INFO] Automatically creating config file for india_buildings_z14.mbtiles
[INFO] Only a basic preview style will be used.
[INFO] See documentation to learn how to create config.json file.
WARN: MBTiles not in "openmaptiles" format. Serving raw data only...
{
  "options": {
    "paths": {
      "root": "/app/node_modules/tileserver-gl-styles",
      "fonts": "fonts",
      "styles": "styles",
      "mbtiles": "/data"
    }
  },
  "styles": {},
  "data": {
    "india_buildings_z14": {
      "mbtiles": "india_buildings_z14.mbtiles"
    }
  }
}
Starting server
Listening at http://0.0.0.0:8080/
Startup complete
```

- live on: https://server.nikhilvj.co.in/buildings1/

## Serving raster tiles
- As per https://tileserver.readthedocs.io/en/latest/config.html , 
- `serve_rendered: true` needs to be there in the style configuration; and it's not there when doing default
- First deployed tileserver-gl without any config specified, it gave vector tiles and a tilejson
- Used a locally run docker instance of https://maputnik.github.io/editor , loaded empty style, added data source as tilejson above
- Added a layer, the source/layer names were auto-added by maputnik.
- Was able to get the buildings appearing in preview in maputnik. Made a style and saved the style json.
- Edited style json to change data source url to "mbtiles://india_buildings_z14.mbtiles" instead of the pbf url that maputnik had saved.
- made config.json point to this style, and kept "serve_rendered": true
- placed both config.json and the style json in the same folder as the .mbtiles file which is mounted as /data in docker command.
- Re-deployed with config:  
```
docker run -d -v$(pwd):/data -p 7100:8080 -p 7101:80 maptiler/tileserver-gl -c config.json --verbose -b 0.0.0.0 -u "https://server.nikhilvj.co.in/buildings1/" 
```
- and now in addition to vector tiles, TileServer GL deployment is giving rasters also: https://server.nikhilvj.co.in/buildings1/
- the config.json and india_buildings_z14_style.json files are uploaded in this repo.


