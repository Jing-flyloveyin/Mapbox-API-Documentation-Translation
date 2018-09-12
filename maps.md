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
`format` | 指定返回瓦片的格式：<table><tr><td>`.grid.json`</td><td>UTFGrid</td></tr><tr><td>`.mvt`</td><td>矢量切片</td></tr><tr><td>`png`</td><td>全彩PNG</td></tr><tr><td>`png32`</td><td>32索引色位 PNG</td></tr><tr><td>`png64`</td><td>32索引色位 PNG</td></tr><tr><td>`png128`</td><td>128索引色位PNG</td></tr><tr><td>`png256`</td><td>256索引色位PNG</td></tr><tr><td>`jpg70`</td><td>70% 品质 JPG</td></tr><tr><td>`jpg80`</td><td>80% 品质 JPG</td></tr><tr><td>`jpg90`</td><td>90% 品质 JPG</td></tr></table> 为了适应不同的网络带宽环境，任何一种图片请求格式都可以被其他格式替换。在性能优先于图片质量的情况下，`jpg70` 和`png32`这种高压缩率的格式就很有用了。<br><br>**注意：** 带有`mapbox.satellite`的瓦片永远以JPEGs的格式传输，即使URL指定其格式为PNG。PNG这种图片格式不能对`mapbox.satellite`这类的摄影图像进行高效的编码。

**请求 style-optimized tiles**

矢量瓦片可以通过在数据请求中包含[style ID](#the-style-object)来进一步优化。如果上述的样式参数已经提供,the sources、[filters](https://www.mapbox.com/mapbox-gl-style-spec/#layer-filter)、 [`minzoom`](https://www.mapbox.com/mapbox-gl-style-spec/#sources-vector-minzoom) 和 [`maxzoom`](https://www.mapbox.com/mapbox-gl-style-spec/#sources-vector-maxzoom)这些样式属性会被解析，地图上不可见部分的数据会从矢量切片上移除。Mapbox GL JS可以通过Mapbox Style JSON去请求托管在Mapbox服务器上的style-optimized矢量切片。

A style-optimized 瓦片请求必要的`style` 查询参数:

查询参数| 描述
--- | ---
`style`<br>(可选) |  `style` 参数分成两部分：样式的ID和这个样式最近被编辑的`timestamp`。这个时间戳参数来自于样式JSON的修改属性，Mapbox Studio创造的所有样式都包含这个参数。

**注意:**  未被使用的图层和要素会从优化样式中移除。如果你打算在使用Mapbox GL JS 或者Mapbox mobile SDK时动态改变样式,扩展过滤器和缩放范围不会同时生效，因为已加载样式中任何不可见的数据，同样也不会在这些数据中。

#### 请求示例

```curl
curl "https://api.mapbox.com/v4/mapbox.mapbox-streets-v7/1/0/0.png?access_token={your_access_token}"

# 获取一张2x的瓦片，这种512x512的瓦片适合像Retina这样的高密度像素展示。
curl "https://api.mapbox.com/v4/mapbox.mapbox-terrain-v2/1/0/0@2x.png?access_token={your_access_token}"

# 获取一张70%质量的JPG编码格式图片
curl "https://api.mapbox.com/v4/mapbox.satellite/3/2/3.jpg70?access_token={your_access_token}"

# 获取一张32PNG格式图片
curl "https://api.mapbox.com/v4/mapbox.mapbox-streets-v7/3/2/3.png32?access_token={your_access_token}"

# 返回一张使用了样式查询参数的style-optimized 瓦片
curl "https://api.mapbox.com/v4/mapbox.mapbox-streets-v7/12/1171/1566.png?style=mapbox://styles/mapbox/streets-v10@00&access_token={your_access_token}"
```

```javascript
// 这个API不能通过JavaScript SDK访问
```

```python
# 这个API不能通过Python SDK访问
```

```bash
# 这个API不能通过Mapbox CLI访问
```

```java
// 这个API不能通过Mapbox Java SDK访问
```

```objc
// 这个API不能通过Mapbox Objective-C libraries访问
```

```swift
//这个API不能通过Mapbox Swift libraries访问
```

### 获取页面形式的 slippy map

```endpoint
GET /v4/{map_id}{/options}.html{#hash}
```

返回一个有slippy map的网页，可以用来分享或者嵌入其他应用。

URL 参数 | 描述
--- | ---
`map_id` | 瓦片数据集的唯一标识，格式为`username.id`。
`options`<br /> (可选) | 地图包含的控制和操作，用逗号分隔：<br><ul><li>`zoomwheel`: 允许鼠标滚轮触发地图缩放</li><li>`zoompan`: 允许地图缩放和平移控制</li><li>`geocoder`: 对产生的slippy map添加地理编码管理</li><li>`share`: 添加一个分享管理</li></ul>

获取页面slippy map的请求，以后会精简成添加一个可选的参数`hash`：

查询参数 | 描述
--- | ---
`hash`<br /> (可选) | 指定地图居中位置的层级和坐标点，格式为`#zoom/lat/lon`。 **注意：** 这个字段在请求中的位置在`access_token`的后面。

#### 请求示例

```curl
# 返回一个有缩放和平移管理、地理编码管理和分享管理的地图。
curl "https://api.mapbox.com/v4/mapbox.mapbox-streets-v7/zoomwheel,zoompan,geocoder,share.html?access_token={your_access_token}"
```

```javascript
// 这个API不能通过JavaScript SDK访问
```

```python
# 这个API不能通过Python SDK访问
```

```bash
# 这个API不能通过Mapbox CLI访问
```

```java
// 这个API不能通过Mapbox Java SDK访问
```

```objc
// 这个API不能通过Mapbox Objective-C libraries访问
```

```swift
// 这个API不能通过Mapbox Swift libraries访问
```

### 获取 TileJSON metadata

```endpoint
GET /v4/{map_id}.json
```

返回瓦片数据集的[TileJSON](https://github.com/mapbox/tilejson-spec/) metadata。TileJSON 对象描述了地图上的一些资源，比如瓦片数据、点位和UTFGrid，以及地图的名称、描述信息和中心点。 

URL 参数 | 描述
--- | ---
`map_id` | 瓦片数据集的唯一标识，格式为`username.id`。

这个端点可以进一步的自定义成一个可选的参数`secure`。

查询 参数 | 描述
--- | ---
`secure`<br /> (可选) | 在默认情况下, 所获取的TileJSON内部的资源URLs（例如在`"tiles"`数组中）将使用HTTP协议。在请求中包含这个查询参数，以接收HTTPS资源url。Include this query parameter in your request to receive HTTPS resource URLs instead.

#### 请求示例

```curl
curl "https://api.mapbox.com/v4/mapbox.satellite.json?access_token={your_access_token}"

# 在获取的TileJSON中请求HTTPS资源url
curl "https://api.mapbox.com/v4/mapbox.satellite.json?secure&access_token={your_access_token}"
```

```javascript
// 这个API不能通过JavaScript SDK访问
```

```python
# 这个API不能通过Python SDK访问
```

```bash
# 这个API不能通过Mapbox CLI访问
```

```java
// 这个API不能通过Java SDK访问
```

```objc
// 这个API不能通过Mapbox Objective-C libraries访问
```

```swift
// 这个API不能通过Mapbox Swift libraries访问
```

#### 响应示例

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

### 获取一个单独的点标注

```endpoint
GET /v4/marker/{name}-{label}+{color}{@2x}.png
```

请求一个单独的点标注，没有相应的背景地图。Request a single marker image without an accompanying background map.

URL 参数 | 描述
--- | ---
`name` | 点标注的形状和大小Marker shape and size. 选项有`pin-s` 和 `pin-l`.
`label`<br /> (可选) | 符合 [Maki v0.5.0](https://github.com/mapbox/maki/blob/v0.5.0/_includes/maki.json) 格式的`icon` 值. 选项有`a`到`z`，`0`到`99`的数字标签，或者可用的Maki图标。 如果请求对象是字母，只会以大写的形式呈现。
`color`<br /> (可选) | A 3- or 6-数字十六进制颜色编码，默认颜色是灰色。
`@2x`<br /> (可选) | 包括请求一个高DPI版本的图像。

#### 请求示例

```curl
# 返回一个高分辨率版本的红色点标注，图标为汽车样式
curl "https://api.mapbox.com/v4/marker/pin-s-car+f44@2x.png?access_token={your_access_token}"

# 返回一个大号点标注，颜色为默认灰色
curl "https://api.mapbox.com/v4/marker/pin-l.png?access_token={your_access_token}"

# 返回一个带有A字母标签的小号蓝色点标注
curl "https://api.mapbox.com/v4/marker/pin-s-a+00f.png?access_token={your_access_token}"
```

```javascript
// 这个API不能通过JavaScript SDK访问
```

```python
# 这个API不能通过Python SDK访问
```

```bash
# 这个API不能通过Mapbox CLI访问
```

```java
// 这个API不能通过Java SDK访问
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

// 同步检索图像，阻塞调用的线程。
// `image` 在iOS、watchOS、tvOS系统上是`UIImage`，在macOS系统上是`NSImage`。
self.imageView.image = snapshot.image;

// 或者，传递一个完成处理器，以便在主线程上异步运行。
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

// 如果Snapshot功能与模块中的另一个类发生冲突，请使用`MapboxStatic.Snapshot`。
let snapshot = Snapshot(
  options: options,
  accessToken: "<#your access token#>")

// 同步检索图像，阻塞调用的线程。
// `image` is a `UIImage` on iOS, watchOS, and tvOS and an `NSImage` on macOS.
imageView.image = snapshot.image

// 或者，传递一个完成处理器，以便在主线程上异步运行。
snapshot.image { (image, error) in
    imageView.image = image
}
```
