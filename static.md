## 静态地图相关设置

Mapbox静态API从[Mapbox Style Specification](https://mapbox.com/mapbox-gl-style-spec)中返回静态地图和栅格切片。

- **静态地图** 是不需要借助映射库或API就可以在网页和移动设备上显示的独立影像。 他们就像是嵌入式地图，但不能交互和控制，反馈的静态地图会保存为PNG文件。
- **栅格切片** 可以作为传统网页的地图数据库，例如 [Mapbox.js](https://mapbox.com/mapbox.js), [Leaflet](https://mapbox.com/help/define-leaflet), 矢量图层等。反馈的栅格切片会保存为JPGE文件，默认尺寸为512×512像素。

对静态API的Swift和Objective-C支持由[MapboxStatic.swift](https://github.com/mapbox/MapboxStatic.swift/) 数据库提供。

利用 [Static API playground](https://www.mapbox.com/help/static-api-playground) 制作一个放缩和平移功能的交互地图。

**限制**

- 静态地图: 默认速率是每分钟600次请求。
- 栅格切片: 默认速率是每分钟2000次请求。
- 超过这些速率限制会导致`HTTP 429`错误。有关速率限制标头的信息可以在 [速率限制](#rate-limits)查看。

如果你需要更高的服务速率，请[联系我们](https://www.mapbox.com/contact/)。

```python
from mapbox import StaticStyle
```

```javascript
const mbxStatic = require('@mapbox/mapbox-sdk/services/static');
const staticClient = mbxStatic({ accessToken: '{your_access_token}' });
```

### 从样式中获取静态地图

```endpoint
GET /styles/v1/{username}/{style_id}/static/{overlay}/{lon},{lat},{zoom},{bearing},{pitch}{auto}/{width}x{height}{@2x} styles:tiles
```

从指定样式中返回PNG格式的静态地图。静态地图端点的使用速率受到访问令牌的限制。默认情况下速率限制为每分钟600次。

地图的位置既可以通过设置 `auto` 参数设定，也可以通过：经度，纬度，缩放等级，旋转角和俯仰角五个参数控制。后两个参数——旋转角和俯仰角是可选的。如果您只需要特定的旋转角而不需要调整俯仰角，俯仰角会默认为 `0`。如果您不需要调整任何参数，它们都会默认为 `0`。如果你设置了 `auto` 参数，就不需要提供任何这些参数。

参数 | 参数描述
---------- | ------------
`username` | 样式所属账户的用户名。
`style_id` | 创建该静态地图的样式的ID。.
`overlay` | 一个或多个逗号分隔的要素，可以在请求时应用于地图的顶部。覆盖层中的要素顺序决定了它们在页面上的z方向上的顺序，位于列表中的最后一项会在最上层（覆盖在列表中其他要素上）并且位于列表中的第一项会在最下面（处于列表中所有要素的最下层）。最终格式可以是 `geojson`, `marker`, 和 `path`的结合体。对于每个选项的更多细节，可以在 [Overlay options section](#overlay-options)中查看。
`lon` | 为静态地图的中心点的经度;位于数字 `-180` 和 `180`之间。
`lat` | 为静态地图的中心点的纬度;位于数字 `-90` 和 `90`之间。
`zoom` | 缩放等级; 位于数字 `0` 和 `20`之间。缩放级别的参数将四舍五入到小数点后两位。
`bearing`<br /> (可选) | 旋转角围绕地图中心旋转地图。取值范围 `0` 和 `360`之间，以十进制计数。设为90将会顺时针旋转地图90°，设为180将会翻转地图。默认旋转度数为`0`。
`pitch`<br /> (可选) | 俯仰角用来设置地图的倾斜度，进而产生正是投影效果。取值范围 `0` 和 `60`之间，单位为度。默认值为 `0`(从地图的正上方正视地图)。
`auto` | 如果添加 `auto` 参数, 视角会结合覆盖层的边界来调整。如果应用 `auto`，它会取代其他五个参数 `lon`, `lat`, `zoom`, `bearing`, `pitch`。
`width` | 图像宽度; 位于数值 `1` 和 `1280` 像素之间。
`height` | 图像高度; 位于数值 `1` 和d `1280` 像素之间。
`@2x`<br /> (可选) | 在`@2x`比例因子下高密度渲染静态地图。

您可以使用以下可选参数进一步细化这个端点的结果:

查询参数 | 参数描述
---------- | ------------
`attribution`<br /> (可选) | 使用布尔值控制图像上的图像归属是否显示。默认为`true`。 **注意:** 如果 `attribution=false`, 会从图像中删除水印属性，您仍然有法律责任为使用OpenStreetMap数据的地图提供所属来源，其中包括大部分 Mapbox 的地图。如果设置 `attribution=false`，法律要求你在 [网页或文件的其他地方注明正确的出处](https://www.mapbox.com/help/attribution/#static--print)。
`logo`<br /> (可选) | 使用布尔值控制图像上是否有 Mapbox 标志。默认为`true`。
`before_layer` (可选) | 一个`string`值用于控制样式中插入`overlay`的位置。所有覆盖层将插入到指定的图层之前。

**叠加选项**<a id="overlay-options"></a>

_GeoJSON_

```
geojson({geojson})
```

参数 | 参数描述
--- | ---
`geojson` | `{geojson}` 文件必须是一个完整可调用的GeoJSON文件。[simplestyle-spec](https://github.com/mapbox/simplestyle-spec) 对于GeoJSON特性的样式将得到渲染。

_Marker_

```
{name}-{label}+{color}({lon},{lat})
```

参数 | 参数描述
--- | ---
`name` | 标记的形状与尺寸。有`pin-s` 和 `pin-l`选项。
`label`<br /> (可选) | 标记符号。选项是一个字母数字标签，从 `a` 到 `z`，从 `0` 到 `99`, 或者是一个可用的 [Maki](https://www.mapbox.com/maki/) 符号。如果被请求的是一个字母，标签将会渲染为大写字母。
`color`<br /> (可选) | 3位或6位十六进制颜色码。
`lon, lat` | 标记中心的位置。当使用非对称标记时，请确保指针位于图像的中心。

_Custom marker_

```
url-{url}({lon},{lat})
```

参数 | 参数描述
--- | ---
`url` | 图像的百分比编码URL。可以是 `PNG` 或 `JPG`格式。
`lon, lat` | 标记中心的位置。当创建一个非对称标记时，比如一个指针，确保指针位于图像的中心。

自定义标记是根据`Expires`和`Cache-Control`标题缓存的。确保至少将这些标头中的一个设置为适当的值，以防止对自定义标记图像的重复请求。

_Path_

```
path-{strokeWidth}+{strokeColor}-{strokeOpacity}+{fillColor}-{fillOpacity}({polyline})
```

[Encoded polylines](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) with a precision of 5 decimal places can be used with the Static API via the `path` parameter.[线编码](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) 通过设置 `path` 参数，精度可达小数点后五位。

参数 | 参数描述
--- | ---
`strokeWidth`<br /> (可选) | 画线的宽度。
`strokeColor`<br /> (可选) | 画线的颜色，3位或6位十六进制颜色代码。
`strokeOpacity`<br /> (可选) | 在`0`(透明)和`1`(不透明)之间的数字，用于设置线条的透明度。
`fillColor`<br /> (可选) | 3位或6位十六进制颜色码填充。
`fillOpacity`<br /> (可选购) | 用`0`(透明)和`1`(不透明)之间的的数值来设置填充透明度。
`polyline` | 编码为URI组件的有效编码线段。

#### 示例

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

### 从样式中获取栅格切片

```endpoint
GET /styles/v1/{username}/{style_id}/tiles/{tilesize}/{z}/{x}/{y}{@2x}
```

从Mapbox Studio样式中获取512x512像素或者256x256像素的栅格切片。返回的栅格切片为JPEG格式。

Mapbox.js 和 Leaflet.js等第三方库通过该端口来渲染Mapbox Studio样式中的栅格切片，具体可参考 [`L.mapbox.styleLayer`](https://www.mapbox.com/mapbox.js/api/v2.4.0/l-mapbox-stylelayer/) 和 [`L.tileLayer`](http://leafletjs.com/reference-1.2.0.html#tilelayer)。默认情况下，访问频率限制为每分钟2000次。

URL 参数 | 参数描述
--- | ---
`username` | 样式所属账户的用户名。
`style_id` | 返回栅格切片的样式ID。
`tilesize`<br> (可选) | 默认大小为512x512像素。（512x512的图像切片与Mapbox Studio经典样式中获取的256x256图像切片相比差了一个缩放层级。例如4级的512x512切片在缩放精度上等于Mapbox Studio经典样式中5级的图像切片。）从该端口获取的256x256切片大小为512x512切片的四分之一，因此对于同一区域地图发送的请求数量是大多数API请求的四倍。
`{z}/{x}/{y}` | 切片坐标，具体描述参考 [Slippy Map Tilenames specification](http://wiki.openstreetmap.org/wiki/Slippy_map_tilenames)。该坐标确定了切片的缩放层级 `{z}`、列号 `{x}`、行号 `{y}`。
`@2x`<br> (可选) | 按放大 `@2x`的倍数来渲染栅格切片，因此切片将会放大至1024x1024像素大小。

#### 示例

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

### 获取地图的WMTS文档

```endpoint
GET /styles/v1/{username}/{style_id}/wmts
```

Mapbox支持 [WMTS](http://www.opengeospatial.org/standards/wmts) 标准，这使得可以在桌面GIS软件和在线GIS软件中使用Mapbox的地图，如ArcMap、QGIS等。

URL 参数 | 参数描述
--- | ---
`username` | 样式所属账户的用户名。
`style_id` | 返回WMTS文档的样式ID。
#### 示例

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
