# OSM Public Transport Downloader
Node.js library to download public transport data from [OpenStreetMaps](https://www.openstreetmap.org/#layers=T) using [Overpass](https://wiki.openstreetmap.org/wiki/Overpass_API) and output it into an extended [GeoJSON](http://geojson.org) file.

## Motivation
This project is part of a set of tools to provide travel data in countries where public transport works on demand and neither bus stops nor timetables exist. Check out https://github.com/trufi-association to see more of our work.

## Why GeoJSON
GeoJSON is a great format to encode geographical data and has a big ecosystem with lots of great tools. GitHub for instance has built-in support for visualizing the contents of a .geojson file. And in essence that is exactly what data from OSM is: Geographical data that was annotated with public transport information. GeoJSON was not made to store all of this information though. That's why we extended standard GeoJSON and output complementary data where it makes sense. The exported data can then be converted into a public transport format of choice in a second step. Use [geojson-to-gtfs](https://github.com/trufi-app/geojson-to-gtfs) to convert it into Google's de facto standard for public transit feeds.

## Usage
```js
// Cochabamba, Bolivia
const bounds = {
    south: -17.57727,  // minimum latitude
    west:  -66.376555, // minimum longitude
    north: -17.276198, // maximum latitude
    east:  -65.96397,  // maximum longitude
};

// Work with returned data
osmToGeojson({ bounds }).then(data => {
    data.geojson // GeoJSON FeatureCollection
    data.stops   // Dictionary that maps OSM node IDs to stop names
    data.log     // Log messages as string
});

// Write data to file system
osmToGeojson({
    bounds,
    outputDir: __dirname + '/out'
}).then(() => {
    // routes.geojson, stops.json, log.txt
});
```

### Examples

The example scripts can be run to export routes data for several city and feature some localization options. There are shortcuts to start the export:

| City | Command |
| ---- | ------- |
| All cities | `npm start` |
| Cochabamba | `npm run bolivia:cochabamba` |
| La Paz | `npm run bolivia:lapaz` |
| Santa Cruz | `npm run bolivia:santacruz` |
| Accra | `npm run ghana:accra` |

The GeoJSON file will be written to `examples/out/<city>/routes.geojson`. You can visualize the data with several tools such as [geojson.io](http://geojson.io/).

![example](/img/routes_geojson_cochabamba.png)

Note: Sometimes the server can reject with `too many requests`. That's why we recommend to run them one by one.

## Options
* `bounds` (object, default `null`) — Defines the OSM bounding box to pull public transport data from. Expects keys `south`, `west`, `north`, `east` with `number` type values.
* `outputDir` (string, default `null`) — When set, writes geojson data, stops and logs into files at the provided location.
* `geojsonFilename` (string, default: `'routes.geojson'`) — Filename for GeoJSON route data when `outputDir` is given.
* `logFilename` (string, default: `'log.txt'`) — Filename for error log when `outputDir` is given.
* `stopsFilename` (string default: `'stops.json'`) — Filename for stops data when `outputDir` is given.
* `stopNameSeparator` (string, default: `' and '`) — Separator that is used when concatenating street names.
* `stopNameFallback` (string, default: `'Unnamed Street'`) — When no street names can be used for a stop, this name is used instead.
* `formatStopName` (function, default: join with stopNameSeparator, use stopNameFallback if empty) — Map names of OSM ways into a stop label
* `mapProperties` (function, default: pass through OSM relation tags) — Maps OSM relation tags into GeoJSON Feature properties.

## Stops data
For each node that is used in one or more routes, a stop name is generated by concatenating the names of all streets together that run through it. Use the `stopName*` settings to change the strings used in the default behaviour. If you need more control, override the `formatStopName` function. The options will be available using `this`.

## Error log
The returned log data contains queries that lead to route errors. You can use [overpass-turbo](http://overpass-turbo.eu/) to run the query and inspect the issue.
