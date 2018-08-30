## Static

The Mapbox Static API returns static maps and raster tiles from styles in the [Mapbox Style Specification](https://mapbox.com/mapbox-gl-style-spec).

- **Static maps** are standalone images that can be displayed on web and mobile devices without the aid of a mapping library or API. They look like an embedded map, but do not have interactivity or controls. The returned static map will be a PNG file.
- **Raster tiles** can be used in traditional web mapping libraries like [Mapbox.js](https://mapbox.com/mapbox.js), [Leaflet](https://mapbox.com/help/define-leaflet), OpenLayers, and others. The returned raster tile will be a JPEG, and will be 512px by 512px by default.

Swift and Objective-C support for the Static API is provided by the [MapboxStatic.swift](https://github.com/mapbox/MapboxStatic.swift/) library.

To build a Static API request by zooming and panning around an interactive map, use the [Static API playground](https://www.mapbox.com/help/static-api-playground).

**Restrictions and limits**

- Static maps: The default rate limit is 600 requests per minute.
- Raster tiles: The default rate limit is 2,000 requests per minute.
- Exceeding these limits will result in an `HTTP 429` response. For information on rate limit headers, see [Rate limits](#rate-limits).

If you require a higher rate limit, [contact us](https://www.mapbox.com/contact/).

```python
from mapbox import StaticStyle
```

```javascript
const mbxStatic = require('@mapbox/mapbox-sdk/services/static');
const staticClient = mbxStatic({ accessToken: '{your_access_token}' });
```

### Retrieve a static map from a style

```endpoint
GET /styles/v1/{username}/{style_id}/static/{overlay}/{lon},{lat},{zoom},{bearing},{pitch}{auto}/{width}x{height}{@2x} styles:tiles
```

Returns a static map from a specified style as a PNG. Use of the static maps endpoint is rate-limited by access token. By default, the rate limit is set to 600 requests per minute.

The position of the map is represented by either the word `auto` or by five numbers: longitude, latitude, zoom, bearing, and pitch. The last two numbers, bearing and pitch, are optional. If you only specify bearing and not pitch, pitch will default to `0`. If you specify neither, they will both default to `0`. If you specify `"auto"`, you should not provide any of these numbers.

Parameter | Description
---------- | ------------
`username` | The username of the account to which the style belongs.
`style_id` | The ID of the style from which to create a static map.
`overlay` | One or more comma-separated features that can be applied on top of the map at request time. The order of features in an overlay dictates their Z-order on the page. The last item in the list will have the highest Z-order (will overlap the other features in the list), and the first item in the list will have the lowest (will underlap the other features). Format can be a mix of `geojson`, `marker`, or `path`. For more details on each option, see the [Overlay options section](#overlay-options).
`lon` | Longitude for the center point of the static map; a number between `-180` and `180`.
`lat` | Latitude for the center point of the static map; a number between `-90` and `90`.
`zoom` | Zoom level; a number between `0` and `20`. Fractional zoom levels will be rounded to two decimal places.
`bearing`<br /> (optional) | Bearing rotates the map around its center. A number between `0` and `360`, interpreted as decimal degrees. 90 rotates the map 90° clockwise, while 180 flips the map. Defaults to `0`.
`pitch`<br /> (optional) | Pitch tilts the map, producing a perspective effect. A number between `0` and `60`, measured in degrees. Defaults to `0` (looking straight down at the map).
`auto` | If `auto` is added, the viewport will fit the bounds of the overlay. If used, `auto` replaces `lon`, `lat`, `zoom`, `bearing`, and `pitch`.
`width` | Width of the image; a number between `1` and `1280` pixels.
`height` | Height of the image; a number between `1` and `1280` pixels.
`@2x`<br /> (optional) | Render the static map at a `@2x` scale factor for high-density displays.

You can further refine the results from this endpoint with the following optional parameters:

Query parameter | Description
---------- | ------------
`attribution`<br /> (optional) | A `boolean` value controlling whether there is attribution on the image. Defaults to `true`. **Note:** If `attribution=false`, the watermarked attribution is removed from the image. You still have a legal responsibility to attribute maps that use OpenStreetMap data, which includes most maps from Mapbox. If you specify `attribution=false`, you are legally required to [include proper attribution elsewhere on the webpage or document](https://www.mapbox.com/help/attribution/#static--print).
`logo`<br /> (optional) | A `boolean` value controlling whether there is a Mapbox logo on the image. Defaults to `true`.
`before_layer` (optional) | A `string` value for controlling where the `overlay` is inserted in the style. All overlays will be inserted before the specified layer.

**Overlay options**<a id="overlay-options"></a>

_GeoJSON_

```
geojson({geojson})
```

Argument | Description
--- | ---
`geojson` | The `{geojson}` argument must be a valid GeoJSON object. [simplestyle-spec](https://github.com/mapbox/simplestyle-spec) styles for GeoJSON features will be respected and rendered.

_Marker_

```
{name}-{label}+{color}({lon},{lat})
```

Argument | Description
--- | ---
`name` | Marker shape and size. Options are `pin-s` and `pin-l`.
`label`<br /> (optional) | Marker symbol. Options are an alphanumeric label `a` through `z`, `0` through `99`, or a valid [Maki](https://www.mapbox.com/maki/) icon. If a letter is requested, it will be rendered in uppercase only.
`color`<br /> (optional) | A 3- or 6-digit hexadecimal color code.
`lon, lat` | The location at which to center the marker. When using an asymmetric marker, make sure that the tip of the pin is at the center of the image.

_Custom marker_

```
url-{url}({lon},{lat})
```

Argument | Description
--- | ---
`url` | A percent-encoded URL for the image. Type can be `PNG` or `JPG`.
`lon, lat` | The location at which to center the marker. When creating an asymmetric marker like a pin, make sure that the tip of the pin is at the center of the image.

Custom markers are cached according to the `Expires` and `Cache-Control` headers. Make sure that at least one of these headers is set to a proper value to prevent repeated requests for the custom marker image.

_Path_

```
path-{strokeWidth}+{strokeColor}-{strokeOpacity}+{fillColor}-{fillOpacity}({polyline})
```

[Encoded polylines](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) with a precision of 5 decimal places can be used with the Static API via the `path` parameter.

Argument | Description
--- | ---
`strokeWidth`<br /> (optional) | A positive number for the line stroke width
`strokeColor`<br /> (optional) | A 3- or 6-digit hexadecimal color code for the line stroke
`strokeOpacity`<br /> (optional) | A number between `0` (transparent) and `1` (opaque) for line stroke opacity
`fillColor`<br /> (optional) | A 3- or 6-digit hexadecimal color code for the fill
`fillOpacity`<br /> (optional) | A number between `0` (transparent) and `1` (opaque) for fill opacity
`polyline` | A valid encoded polyline encoded as a URI component

#### Example request

```curl
# Retrieve a map at -122.4241 longitude, 37.78 latitude,
# zoom 14.24, bearing 0, and pitch 60. The map
# will be 600 pixels wide and 600 pixels high
curl "https://api.mapbox.com/styles/v1/mapbox/streets-v10/static/-122.4241,37.78,14.25,0,60/600x600?access_token={your_access_token}"

# Retrieve a map at 0 longitude, 10 latitude, zoom 3,
# and bearing 20. Pitch will default to 0.
curl "https://api.mapbox.com/styles/v1/mapbox/streets-v10/static/0,10,3,20/600x600?access_token={your_access_token}"

# Retrieve a map at 0 longitude, 0 latitude, zoom 2.
# Bearing and pitch default to 0.
curl "https://api.mapbox.com/styles/v1/mapbox/streets-v10/static/0,0,2/600x600?access_token={your_access_token}"

# Retrieve a map with a custom marker overlay
curl "https://api.mapbox.com/styles/v1/mapbox/streets-v10/static/url-https%3A%2F%2Fwww.mapbox.com%2Fimg%2Frocket.png(-76.9,38.9)/-76.9,38.9,15/1000x1000?access_token={your_access_token}"

# Retrieve a map with a GeoJSON overlay
curl "https://api.mapbox.com/styles/v1/mapbox/streets-v10/static/geojson(%7B%22type%22%3A%22Point%22%2C%22coordinates%22%3A%5B-73.99%2C40.7%5D%7D)/-73.99,40.70,12/500x300?access_token={your_access_token}"

# Retrieve a map with 2 points and a polyline overlay,
# with its center point automatically determined with `auto`
curl "https://api.mapbox.com/styles/v1/mapbox/streets-v10/static/pin-s-a+9ed4bd(-122.46589,37.77343),pin-s-b+000(-122.42816,37.75965),path-5+f44-0.5(%7DrpeFxbnjVsFwdAvr@cHgFor@jEmAlFmEMwM_FuItCkOi@wc@bg@wBSgM)/auto/500x300?access_token={your_access_token}"
```

```javascript
staticClient
  .getStaticImage({
    ownerId: 'mapbox',
    styleId: 'streets-v10',
    width: 200,
    height: 300,
    coordinates: [12, 13],
    zoom: 4
  })
  .send()
  .then(response => {
    const image = response.body;
  });

staticClient
  .getStaticImage({
    ownerId: 'mapbox',
    styleId: 'streets-v10',
    width: 200,
    height: 300,
    coordinates: [12, 13],
    zoom: 3,
    overlays: [
      // Simple markers.
      {
        marker: {
          coordinates: [12.2, 12.8]
        }
      },
      {
        marker: {
          size: 'large',
          coordinates: [14, 13.2],
          label: 'm',
          color: '#000'
        }
      },
      {
        marker: {
          coordinates: [15, 15.2],
          label: 'airport',
          color: '#ff0000'
        }
      },
      // Custom marker
      {
        marker: {
          coordinates: [10, 11],
          url:
            'https://upload.wikimedia.org/wikipedia/commons/6/6f/0xff_timetracker.png'
        }
      }
    ]
  })
  .send()
  .then(response => {
    const image = response.body;
  });

// To get the URL instead of the image, create a request
// and get its URL without sending it.
const request = staticClient
  .getStaticImage({
    ownerId: 'mapbox',
    styleId: 'streets-v10',
    width: 200,  
    height: 300,
    coordinates: [12, 13],
    zoom: 4
  });
const staticImageUrl = request.url();
// Now you can open staticImageUrl in a browser.
```

```python
response = service.image(
    username='mapbox',
    style_id='streets-v9',
    lon=-122.7282, lat=45.5801, zoom=12)
# if features are provided the map image will be centered on them
portland = {
    'type': 'Feature',
    'properties': {'name': 'Portland, OR'},
    'geometry': {
        'type': 'Point',
        'coordinates': [-122.7282, 45.5801]}}
bend = {
    'type': 'Feature',
    'properties': {'name': 'Bend, OR'},
    'geometry': {
        'type': 'Point',
        'coordinates': [-121.3153, 44.0582]}}
response = service.image(
    username='mapbox',
    style_id='streets-v9',
    features=[portland, bend])
```

```bash
# This API cannot be accessed with Mapbox CLI
```

```java
MapboxStaticImage staticImage = MapboxStaticMap.builder()
  .accessToken("{your_access_token}")
  .styleId(Constants.MAPBOX_STYLE_STREETS)
  .cameraPoint(Point.fromLngLat(imageTarget.getLongitude(), imageTarget.getLatitude())))
  .cameraZoom(imageZoom)
  .cameraPitch(60)
  .cameraBearing(45)
  .width(width)
  .height(height)
  .retina(true)
  .build();
```

```objc
@import MapboxStatic;

NSURL *styleURL = [NSURL URLWithString:@"mapbox://styles/mapbox/streets-v10"];
CLLocationCoordinate2D coordinate = CLLocationCoordinate2DMake(37.78, -122.4241);
MBSnapshotCamera *camera = [MBSnapshotCamera cameraLookingAtCenterCoordinate:coordinate zoomLevel:14.25];
camera.pitch = 60;
// Dimensions are measured in resolution-independent points.
CGSize size = CGSizeMake(600, 600);
MBSnapshotOptions *options = [[MBSnapshotOptions alloc] initWithStyleURL:styleURL camera:camera size:size];

MBSnapshot *snapshot = [[MBSnapshot alloc] initWithOptions:options accessToken:@"<#your access token#>"];

// Retrieve the image synchronously, blocking the calling thread.
// `image` is a `UIImage` on iOS, watchOS, and tvOS and an `NSImage` on macOS.
self.imageView.image = snapshot.image;

// Alternatively, pass a completion handler to run asynchronously on the main thread.
[snapshot imageWithCompletionHandler:^(UIImage * _Nullable image, NSError * _Nullable error) {
    self.imageView.image = image;
}];
```

```swift
import MapboxStatic

let camera = SnapshotCamera(
    lookingAtCenter: CLLocationCoordinate2D(latitude: 37.78, longitude: -122.4241),
    zoomLevel: 14.25)
camera.pitch = 60
let options = SnapshotOptions(
  styleURL: styleURL,
  camera: camera,
  // Dimensions are measured in resolution-independent points.
  size: CGSize(width: 200, height: 200))

// If Snapshot conflicts with another class in your module, use `MapboxStatic.Snapshot`.
let snapshot = Snapshot(
  options: options,
  accessToken: "<#your access token#>")

// Retrieve the image synchronously, blocking the calling thread.
// `image` is a `UIImage` on iOS, watchOS, and tvOS and an `NSImage` on macOS.
imageView.image = snapshot.image

// Alternatively, pass a completion handler to run asynchronously on the main thread.
snapshot.image { (image, error) in
    imageView.image = image
}
```

### Retrieve raster tiles from styles

```endpoint
GET /styles/v1/{username}/{style_id}/tiles/{tilesize}/{z}/{x}/{y}{@2x}
```

Retrieve 512x512 pixel or 256x256 pixel raster tiles from a Mapbox Studio style. The returned raster tile will be a JPEG.

Libraries like Mapbox.js and Leaflet.js use this endpoint to render raster tiles from a Mapbox Studio style with [`L.mapbox.styleLayer`](https://www.mapbox.com/mapbox.js/api/v2.4.0/l-mapbox-stylelayer/) and [`L.tileLayer`](http://leafletjs.com/reference-1.2.0.html#tilelayer). By default, the rate limit is set to 2,000 requests per minute.

URl parameter | Description
--- | ---
`username` | The username of the account to which the style belongs.
`style_id` | The ID of the style from which to return a raster tile.
`tilesize`<br> (optional) | Default is 512x512 pixels. (512x512 image tiles are offset by 1 zoom level compared to 256x256 tiles from Mapbox Studio Classic styles. For example, 512x512 tiles at zoom level 4 are equivalent to Mapbox Studio Classic styles tiles at zoom level 5.) 256x256 tiles from the endpoint are one quarter of the size of 512x512 tiles. Therefore, they require 4 times as many API requests and accumulate 4 times as many map views to render the same area.
`{z}/{x}/{y}` | The tile coordinates as described in the [Slippy Map Tilenames specification](http://wiki.openstreetmap.org/wiki/Slippy_map_tilenames). They specify the tile's zoom level `{z}`, column `{x}`, and row `{y}`.
`@2x`<br> (optional) | Render the raster tile at a `@2x` scale factor, so tiles are scaled to 1024x1024 pixels.

#### Example request

```curl
# Returns a default 512x512 pixel tile
curl "https://api.mapbox.com/styles/v1/mapbox/streets-v10/tiles/1/1/0?access_token={your_access_token}"

# Returns a 256x256 pixel tile
curl "https://api.mapbox.com/styles/v1/mapbox/streets-v10/tiles/256/1/1/0?access_token={your_access_token}"

# Returns a 1024x1024 pixel tile
curl "https://api.mapbox.com/styles/v1/mapbox/streets-v10/tiles/512/1/1/0@2x?access_token={your_access_token}"
```

```javascript
// This API cannot be accessed with the JavaScript SDK
```

```python
# This API cannot be accessed with the Python SDK
```

```bash
# This API cannot be accessed with Mapbox CLI
```

```java
// This API cannot be accessed with the Mapbox Java libraries
```

```objc
// This API cannot be accessed with the Mapbox Objective-C libraries
```

```swift
// This API cannot be accessed with the Mapbox Swift libraries
```

### Retrieve a map's WMTS document

```endpoint
GET /styles/v1/{username}/{style_id}/wmts
```

Mapbox supports access via the [WMTS](http://www.opengeospatial.org/standards/wmts) standard, which lets you use maps with desktop and online GIS software like ArcMap and QGIS.

URL parameter | Description
--- | ---
`username` | The username of the account to which the style belongs.
`style_id` | The ID of the style for which to return a WMTS document.

#### Example request

```curl
curl "https://api.mapbox.com/styles/v1/mapbox/streets-v10/wmts?access_token={your_access_token}"
```

```javascript
// This API cannot be accessed with the JavaScript SDK
```

```python
# This API cannot be accessed with the Python SDK
```

```bash
# This API cannot be accessed with Mapbox CLI
```

```java
// This API cannot be accessed with Mapbox Java libraries
```

```objc
// This API cannot be accessed with the Mapbox Objective-C libraries
```

```swift
// This API cannot be accessed with the Mapbox Swift libraries
```
