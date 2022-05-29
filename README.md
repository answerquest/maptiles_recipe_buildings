# maptiles_recipe_buildings
recipes to serve vector and raster map tiles of datasets by https://github.com/microsoft/GlobalMLBuildingFootprints

## Latest status: 
- as of: 2022-05-29
- Vector map tiles layer working, see: https://server.nikhilvj.co.in/buildings1/ (Tileserver GL deployment), url: `https://server.nikhilvj.co.in/buildings1/data/buildings-z13/{z}/{x}/{y}.pbf`
- Raster layer: not yet!

## Converting to mbtiles using Tippecanoe
- https://github.com/mapbox/tippecanoe 
- Command:
```
tippecanoe -zg -o out.mbtiles --drop-densest-as-needed India.geojsonl
```
- Created a .mbtiles file in about 6 hrs. Did not take too much RAM as it didn't have to load the entire data into memory.

## Serving vector tiles
- loading the generated .mbtiles straight into tileserver-gl without any config is working:
```
docker run --rm -it -v $(pwd):/data -p 7100:8080 -p 7101:80 maptiler/tileserver-gl --verbose -b 0.0.0.0 -u "https://server.nikhilvj.co.in/buildings1/"
```
- output:
```
Starting tileserver-gl v3.1.1
No MBTiles specified, using buildings-z13.mbtiles
[INFO] Automatically creating config file for buildings-z13.mbtiles
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
    "buildings-z13": {
      "mbtiles": "buildings-z13.mbtiles"
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
- Plus, note the `WARN: MBTiles not in "openmaptiles" format. Serving raw data only...` line.
- And so, the rendered .png or .webp paths as shown on https://tileserver.readthedocs.io/en/latest/endpoints.html are not working in the current tileserver-gl deployment; it is only giving vector .pbf tiles.

