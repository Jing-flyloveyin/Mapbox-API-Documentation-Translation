## Maps

The Mapbox Maps API 支持通过images、TileJSON或者平滑的嵌入式页面地图的形式获取矢量和影像的瓦片数据集。

如果你使用过[Mapbox GL JS](https://www.mapbox.com/mapbox-gl-js/api/)、 [Mapbox.js](https://www.mapbox.com/mapbox.js/) 或者其他库比如[Leaflet](https://www.mapbox.com/mapbox.js/example/v1.0.0/plain-leaflet/)，说明你已经使用过 Maps API，如果要设计或者使用Mapbox Maps，就不需要阅读这份指南。相反, 这份文档对那些想通过编程的方式来理解这些资源的软件开发人员很有意义。

**约束和限制**

- Use of the Mapbox Maps API endpoint is rate limited based on your user plan. 默认设置是每分钟100000次请求数。The default is 100,000 requests per minute.
- 超过用户计划的每分钟请求数会出现“HTTP 429 Too Many Requests”的响应。Exceeding your user plan's number requests per minute will result in an `HTTP 429 Too Many Requests` response.
- 关于速率限制头文件的信息，请参考[Rate limits](#rate-limits) 章节。

如果你需要提高速率限制，[请联系我们](https://www.mapbox.com/contact/)。

### 瓦片获取

```endpoint
GET /v4/{map_id}/{zoom}/{x}/{y}{@2x}.{format}
```

返回一张栅格瓦片、矢量瓦片或者是指定格式的UTFGrid图。

响应结果是一张指定格式的图片，出于性能考虑，带有`max-age`头部标识值且值为12小时的图像瓦片将在之后的时间传输。The response is an image tile in the specified format. For performance, image tiles are delivered with a `max-age` header value set 12 hours in the future.

URL 参数 | 描述
--- | --
`map_id` | Unique identifier for the tileset 这种格式`username.id`的瓦片数据集的唯一标识。若要组成多个瓦片数据集，请使用分号分隔参数且最多有15个`map_id`。
`zoom` | 指定瓦片的缩放层级，具体说明参见[Slippy Map Tilenames](http://wiki.openstreetmap.org/wiki/Slippy_map_tilenames)。
`{x}/{y}` | 指定瓦片的列 `{x}` 和 行 `{y}`, 具体说明参见[Slippy Map Tilenames](http://wiki.openstreetmap.org/wiki/Slippy_map_tilenames)。
`@2x`<br>(可选) | 请求有更高DPI（分辨率）版本的图像。
`format` | 指定返回瓦片的格式：<table><tr><td>`.grid.json`</td><td>UTFGrid</td></tr><tr><td>`.mvt`</td><td>矢量切片</td></tr><tr><td>`png`</td><td>全彩PNG</td></tr><tr><td>`png32`</td><td>32索引色位 PNG</td></tr><tr><td>`png64`</td><td>32索引色位 PNG</td></tr><tr><td>`png128`</td><td>128索引色位PNG</td></tr><tr><td>`png256`</td><td>256 color indexed PNG</td></tr><tr><td>`jpg70`</td><td>70% quality JPG</td></tr><tr><td>`jpg80`</td><td>80% quality JPG</td></tr><tr><td>`jpg90`</td><td>90% quality JPG</td></tr></table> The `format` of any image request can be replaced by any of these formats to adjust image quality for different bandwidth requirements. Higher-compression formats like `jpg70` or `png32` can be useful to favor performance over image quality.<br><br>**Note:** Tiles that include `mapbox.satellite` are always delivered as JPEGs, even if the URL specifies PNG. The PNG format can't efficiently encode photographic images like those used by `mapbox.satellite`.

**Request style-optimized tiles**

Vector tiles can be further optimized by including a [style ID](#the-style-object) with the tile request. If the style parameter is provided, the sources, [filters](https://www.mapbox.com/mapbox-gl-style-spec/#layer-filter), [`minzoom`](https://www.mapbox.com/mapbox-gl-style-spec/#sources-vector-minzoom), and [`maxzoom`](https://www.mapbox.com/mapbox-gl-style-spec/#sources-vector-maxzoom) properties of that style are analyzed, and data that won't be visible on the map is removed from the vector tile. Mapbox GL JS can request style-optimized vector tiles that are hosted on Mapbox with a Mapbox Style JSON.

A style-optimized tile request requires the `style` query parameter:

Query parameter | Description
--- | ---
`style`<br>(optional) | The `style` parameter is broken into two parts, the style ID and the style's recently edited `timestamp`. The timestamp parameter comes from the style JSON's modified property, which is included with any style created with Mapbox Studio.

**Note:** Unused layers and features are removed from optimized styles. If you plan to dynamically change the style at runtime using Mapbox GL JS or a Mapbox mobile SDK, broadening filters and zoom ranges won't work the same way since any data that isn't visible with the loaded style also won't be included in the data.

#### Example request

```curl
curl "https://api.mapbox.com/v4/mapbox.mapbox-streets-v7/1/0/0.png?access_token={your_access_token}"

# Retrieve a 2x tile; this 512x512 tile is appropriate for high-density displays like Retina
curl "https://api.mapbox.com/v4/mapbox.mapbox-terrain-v2/1/0/0@2x.png?access_token={your_access_token}"

# Retrieve a tile with 70% quality JPG encoding
curl "https://api.mapbox.com/v4/mapbox.satellite/3/2/3.jpg70?access_token={your_access_token}"

# Retrieve a tile with 32 color indexed PNG
curl "https://api.mapbox.com/v4/mapbox.mapbox-streets-v7/3/2/3.png32?access_token={your_access_token}"

# Return a style-optimized tile using the style query parameter
curl "https://api.mapbox.com/v4/mapbox.mapbox-streets-v7/12/1171/1566.png?style=mapbox://styles/mapbox/streets-v10@00&access_token={your_access_token}"
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
// This API cannot be accessed with the Mapbox Java SDK
```

```objc
// This API cannot be accessed with the Mapbox Objective-C libraries
```

```swift
// This API cannot be accessed with the Mapbox Swift libraries
```

### Retrieve an HTML slippy map

```endpoint
GET /v4/{map_id}{/options}.html{#hash}
```

Returns HTML for a slippy map that can be used for sharing or embedding.

URL parameter | Description
--- | ---
`map_id` | Unique identifier for the tileset in the format `username.id`
`options`<br /> (optional) | A comma-separated list of controls and map behaviors to be included in the map:<br><ul><li>`zoomwheel`: Enable zooming with the mouse wheel</li><li>`zoompan`: Enable zoom and pan controls</li><li>`geocoder`: Add a geocoder control to the result slippy map</li><li>`share`: Add a share control</li></ul>

A request to retrieve an HTML slippy map can be further refined by adding the optional `hash` parameter:

Query parameter | Description
--- | ---
`hash`<br /> (optional) | Specify a zoom level and location for the map to center on, in the format `#zoom/lat/lon`. **Note:** This hash is placed after the `access_token` in the request.

#### Example request

```curl
# Returns a map with zoom and pan controls, a geocoder, and a share control
curl "https://api.mapbox.com/v4/mapbox.mapbox-streets-v7/zoomwheel,zoompan,geocoder,share.html?access_token={your_access_token}"
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
// This API cannot be accessed with the Mapbox Java SDK
```

```objc
// This API cannot be accessed with the Mapbox Objective-C libraries
```

```swift
// This API cannot be accessed with the Mapbox Swift libraries
```

### Retrieve TileJSON metadata

```endpoint
GET /v4/{map_id}.json
```

Returns [TileJSON](https://github.com/mapbox/tilejson-spec/) metadata for a tileset. The TileJSON object describes a map's resources, like tiles, markers, and UTFGrid, as well as its name, description, and centerpoint.

URL parameter | Description
--- | ---
`map_id` | Unique identifier for the tileset in the format `username.id`.

This endpoint can be further customized with the optional `secure` parameter:

Query parameter | Description
--- | ---
`secure`<br /> (optional) | By default, resource URLs in the retrieved TileJSON (such as in the  `"tiles"` array) will use the HTTP scheme. Include this query parameter in your request to receive HTTPS resource URLs instead.

#### Example request

```curl
curl "https://api.mapbox.com/v4/mapbox.satellite.json?access_token={your_access_token}"

# Request HTTPS resource URLs in the retrieved TileJSON
curl "https://api.mapbox.com/v4/mapbox.satellite.json?secure&access_token={your_access_token}"
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
// This API cannot be accessed with the Mapbox Java SDK
```

```objc
// This API cannot be accessed with the Mapbox Objective-C libraries
```

```swift
// This API cannot be accessed with the Mapbox Swift libraries
```

#### Example response

```json
{
  "attribution": "<a href=\"https://www.mapbox.com/about/maps/\" target=\"_blank\">&copy; Mapbox</a> <a href=\"http://www.openstreetmap.org/about/\" target=\"_blank\">&copy; OpenStreetMap</a> <a class=\"mapbox-improve-map\" href=\"https://www.mapbox.com/map-feedback/\" target=\"_blank\">Improve this map</a> <a href=\"https://www.digitalglobe.com/\" target=\"_blank\">&copy; DigitalGlobe</a>",
  "autoscale": true,
  "bounds": [-180, -85, 180, 85],
  "cacheControl": "max-age=43200,s-maxage=604800",
  "center": [0, 0, 3],
  "created": 1358310600000,
  "description": "",
  "id": "mapbox.satellite",
  "maxzoom": 19,
  "minzoom": 0,
  "modified": 1446150592060,
  "name": "Mapbox Satellite",
  "private": false,
  "scheme": "xyz",
  "tilejson": "2.0.0",
  "tiles": [
    "http://a.tiles.mapbox.com/v4/mapbox.satellite/{z}/{x}/{y}.png",
    "http://b.tiles.mapbox.com/v4/mapbox.satellite/{z}/{x}/{y}.png"
  ],
  "webpage": "http://a.tiles.mapbox.com/v4/mapbox.satellite/page.html"
}
```

### Retrieve a standalone marker

```endpoint
GET /v4/marker/{name}-{label}+{color}{@2x}.png
```

Request a single marker image without an accompanying background map.

URL Parameter | Description
--- | ---
`name` | Marker shape and size. Options are `pin-s` and `pin-l`.
`label`<br /> (optional) | A [Maki v0.5.0](https://github.com/mapbox/maki/blob/v0.5.0/_includes/maki.json) `icon` value. Options are an alphanumeric label `a` through `z`, `0` through `99`, or a valid Maki icon. If a letter is requested, it will be rendered in uppercase only.
`color`<br /> (optional) | A 3- or 6-digit hexadecimal color code. The default color is gray.
`@2x`<br /> (optional) | Include to request a high DPI version of the image.

#### Example request

```curl
# Returns a small red marker that contains a car icon, high DPI version
curl "https://api.mapbox.com/v4/marker/pin-s-car+f44@2x.png?access_token={your_access_token}"

# Returns a large default gray marker
curl "https://api.mapbox.com/v4/marker/pin-l.png?access_token={your_access_token}"

# Returns a small blue marker labeled A
curl "https://api.mapbox.com/v4/marker/pin-s-a+00f.png?access_token={your_access_token}"
```

```javascript
// This method cannot be accessed with the JavaScript SDK
```

```python
# This method cannot be accessed with the Python SDK
```

```bash
# This method cannot be accessed with Mapbox CLI
```

```java
// This API cannot be accessed with the Mapbox Java SDK
```

```objc
@import MapboxStatic;

MBMarkerOptions *options = [[MBMarkerOptions alloc] initWithSize:MBMarkerSizeSmall iconName:@"cafe"];
#if TARGET_OS_IOS || TARGET_OS_TV || TARGET_OS_WATCH
    options.color = [UIColor brownColor];
#elif TARGET_OS_MAC
    options.color = [NSColor brownColor];
#endif
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

let options = MarkerOptions(
  size: .small,
  iconName: "cafe")
options.color = .brown

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
