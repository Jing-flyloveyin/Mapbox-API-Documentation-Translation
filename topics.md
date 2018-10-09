## 阅读这篇文档

这篇文档是由如[Geocoding](#geocoding)或[Uploads](#uploads)这样一组相关功能的API和相关服务端点构成，每个服务端点是一个API中用于执行某个操作并位于特定URL的特定方法   

本文档中的每个服务端点使用如下几个部分描述:   

* HTTP方法: 包含GET, POST, PUT, PATCH和DELETE   
* 路径: 例如`/geocoding/v5/{mode}/{query}.json`   
* URL参数: 这些参数是服务端点路径中括号括起来的部分,
  像例子中的`{mode}`.
* 查询参数: 包含在一个有 _Option_ 请求头的表中, 并且这些参数被添加到请求的查询字符部分.
* 令牌作用域, 如果在需要一个令牌的情况下.  

所有文档中提到的URL有以下基本路径 
`https://api.mapbox.com`. 
这个基本路径在服务端点路径之前. 在这个例子中，你需要把 `https://api.mapbox.com`和 `/geocoding/v5/{mode}/{query}.json`组合得到请求的URL `https://api.mapbox.com/geocoding/v5/{mode}/{query}.json`.

在这个服务端点中, `{mode}`和 `{query}`是URL参数. 在一个请求中，你将用真实值去替换对应的占位符: 例如，你应该选择 `mapbox.places`作为你的模式, `Chester`作为你要查询的目标, 并得到这样的URL `https://api.mapbox.com/geocoding/v5/mapbox.places/Chester.json`  

查询参数是用[查询字符串编码](https://en.wikipedia.org/wiki/Query_string)来添加到URL的结尾.如果你想把 `country` 这个请求参数添加到这个Geocoding请求中，你应该把查询字符串 `?country=us`加到URL的结尾, 得到`https://api.mapbox.com/geocoding/v5/mapbox.places/Chester.json?country=us`.  

所有的服务端点需要一个访问令牌, 这个令牌是作为一个查询参数被提供的.   
所以最终你要构造的geocoding请求会是这样`https://api.mapbox.com/geocoding/v5/mapbox.places/Chester.json?country=us&access_token=pk.my-token-value`.
下一章节包括你将如何获得和使用访问令牌.

```http
https://api.mapbox.com
```

## 访问令牌

访问Mapbox的web服务需要一个[访问令牌](https://mapbox.com/help/define-access-token), 这个访问令牌会连接API请求到你的账户. 本文档中的示例请求不包括访问令牌: 你需要使用`access_token`查询选项来提供一个或者在SDK或者库中指定一个令牌.

在你的[账户面板](https://www.mapbox.com/account)中可以得到默认访问令牌. 你也可以在你的[访问令牌页](https://www.mapbox.com/account/access-tokens/)或者使用[Tokens API](#tokens)管理和创建附加的令牌.

#### 访问令牌用例

```http
https://api.mapbox.com/{endpoint}?access_token={your_access_token}
```

当创建一个新的访问令牌时, 您可以选择添加一个或多个**作用域**. 
每个作用域为令牌添加了不同的权限来用于
访问受限API. 在整个文档中, 我们指定了访问每个服务端点所必需的作用域.

## 版本控制

每个 Mapbox API 都可基于 URL 指定所期望的 api 版本, 并可使得该 API 版本控制独立于其他 api 递增。

#### 版本控制例子

```http
https://api.mapbox.com/{api}/{version}
```

_[Maps API](#maps) 是一个例外: 他的 endpoint 是以 `/v4/{map_id}` 为前缀，而不是将版本号放在 api 
名称后，这个特例将在下一个 api 版本中修复。_

我们始终建议您使用最新版本的 API。

以下改动将会被认定为具有向后兼容性，当其发布时我们将不会更新版本号:

- JSON 对象中增加新的属性。
- 更改单个列表请求中返回的项数。
- 更改速率限制的阈值。
- 更改 API 自动生成的 ID 标识符的结构或长度。
- 更改错误信息。

以下改动将会被认定为不具有向后兼容性，当其发布时我们将会更新版本号:

- JSON 对象移除一些属性。
- 更改 API 的 url 结构

当决定某些功能将要被弃用[deprecated], 我们至少在生效前 90 天通过电子邮件通知您。当我们检测到您使用的 API 
中存在弃用功能时，我们也会给您发送电子邮件。

## 速率限制

Mapbox APIs 将会对每个 endpoint 的请求量进行速率限制。如果您超出了速率限制，您的请求将会被拒接，并会收到
API 的请求相应 `HTTP 429 Too Many Requests` 。

| Header | 描述 |
| --- | --- |
| `X-Rate-Limit-Interval` | 速率限制的间隔长度。（以秒为单位） |
| `X-Rate-Limit-Limit` | 每个速率限制间隔期中可进行的最大请求数。|
| `X-Rate-Limit-Reset` | 一个 unix 时间戳，在当前时间间隔且速率限制计数器被重置时的时间。|

## CORS

Mapbox web 服务支持不受域限制的 [跨源请求(Cross-Origin Requests)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS) 。 为了支持 IE 8 和 IE9，请使用相关的库 例如 
[corslite](https://github.com/mapbox/corslite/) ，让请求退回到 XDomainRequest,

## Retina

```http
# 一张400x200静态地图
https://api.mapbox.com/v4/mapbox.dark/-76.9,38.9,5/400x200.png?access_token={your_access_token}

# 相同的静态地图Retina
# 显示：这个图像将是800x400
# 像素，但是显示相同的内容
# 比例缩小后看起来也一样
https://api.mapbox.com/v4/mapbox.dark/-76.9,38.9,5/400x200@2x.png?access_token={your_access_token}
```

Mapbox支持所有提供图像的APIs上的Retina图像输出。
在URL上的文件扩展名之前添加`@2x`，以双倍比例请求图像。
例如， 一张带有`@2x`的256×256像素的地图瓦片将是
512×512像素，但显示相同的内容。
当在页面上显示时，图像的大小仍将为256×256像素，
但原始的4个像素将以屏幕单元表示1个像素。

URL的`@2x`部分先于整个格式，
因此以`.png`结尾的URL将以`@2x.png`作为Retina图像的结尾。

唯一不能在Retina比例上使用的资源是TileMill上传的tilesets
或MBTiles。

## HTTPS

我们推荐通过HTTPS来访问Mapbox。
除Maps API外，通过HTTP启动的请求会自动升级为HTTPS。

## Pagination

```curl
Link: <https://api.mapbox.com/uploads/v1/1454024795582?start=cijywvxlm004rtikohhqf99jv&limit=100>; rel="next"
```

分页允许您使用多个请求从API列出许多对象。
在接收到一页对象之后，
可以使用响应的 [`Link` header](http://tools.ietf.org/html/rfc5988) 中的 `下一个` 链接关系来请求下一页。
可以重复此过程，
直到服务器发送没有`链接`头或没有`下一个`链接关系的响应，
这表示集合的结束。

您的应用程序 *必须* 使用`链接`标头进行分页，
而不是构建您自己的URLs，因为用于分页的特定URLs可能随时更改。
[Python requests library](http://docs.python-requests.org/en/master/user/advanced/#link-headers),
和
[link-header-parser module for JavaScript](https://github.com/thlorenz/parse-link-header)
能解析链接头。链接头符合 [RFC 5988 specifications](http://tools.ietf.org/html/rfc5988) 规范。

在[Datasets](#datasets), [Uploads](#uploads), [Styles](#styles), [Tilesets](#tilesets), 和 [Tokens](#tokens) 列表终端点上支持分页。

| 查询参数 | 说明 |
| --- | --- |
| `限制` | 要返回对象的*最大*数。API将尝试返回对象的请求数，但是接收更少的对象并不一定意味着集合的结束。接收没有`链接`头或没有`下一个`链接关系的响应是确定何时处于集合末尾的唯一方法。 |

## Dates

API返回的大多数日期和时间都
以[RFC 3339](https://tools.ietf.org/html/rfc3339) 格式表示，可以通过
[JavaScript Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)
构造函数，[Python arrow library](https://github.com/crsmithdev/arrow)，
以及许多其他库和语言进行解析。 

```javascript
var parsedDate = new Date('2014-11-21T19:41:10.000Z');
```

```python
import arrow
arrow.get('2014-11-21T19:41:10.000Z')
<Arrow [2014-11-21T19:41:10+00:00]>
```

```objc
@import Foundation;

NSDateFormatter *RFC3339DateFormatter = [[NSDateFormatter alloc] init];
RFC3339DateFormatter.locale = [NSLocale localeWithLocaleIdentifier:@"en_US_POSIX"];
RFC3339DateFormatter.dateFormat = @"yyyy-MM-dd'T'HH:mm:ssZZZZZ";
RFC3339DateFormatter.timeZone = [NSTimeZone timeZoneForSecondsFromGMT:0];

NSDate *date = [RFC3339DateFormatter dateFromString:@"2014-11-21T19:41:10.000Z"];
```

```swift
import Foundation

let rfc3339DateFormatter = DateFormatter()
rfc3339DateFormatter.locale = Locale(identifier: "en_US_POSIX")
rfc3339DateFormatter.dateFormat = "yyyy-MM-dd'T'HH:mm:ssZZZZZ"
rfc3339DateFormatter.timeZone = TimeZone(secondsFromGMT: 0)

let date = rfc3339DateFormatter.date(from: "2014-11-21T19:41:10.000Z")
```

此规则的唯一例外是 [Retrieve TileJSON metadata](#retrieve-tilejson-metadata)
终端点，它将`创建`和`修改`的属性作为 [Unix time](https://en.wikipedia.org/wiki/Unix_time) 返回。

```objc
@import Foundation;

NSDate *date = [NSDate dateWithTimeIntervalSince1970:1416598870000];
```

```swift
import Foundation

let date = Date(timeIntervalSince1970: 1_416_598_870_000)
```

## Coordinates

如果地理坐标提供给Mapbox API，
则应按照`经度，纬度`的顺序对其进行格式化，并在WGS84坐标系中指定为十进制度。
这种模式符合现有标准，包括GeoJSON和KML。

Mapbox APIs use GeoJSON formatting wherever possible to represent Mapbox APIs尽可能使用GeoJSON格式来表示
geospatial data. 地理空间数据。方向，地图匹配，地理编码和数据集APIs The Directions, Map Matching, Geocoding, and Datasets APIs
都返回GeoJSON格式的响应，all return GeoJSON-formatted responses, and the Upload and Map Matching APIs
accept GeoJSON input.上传和地图匹配APIs都接受GeoJSON输入。

唯一例外的`经度，纬度`顺序是多段线，
在静态（经典）叠加和路线响应支持。 当指定多段线
输入或输出时，多段线内容应遵循Google编码多段线格式，
该格式指定`纬度，经度`顺序。

Mapbox Swift库使用了核心位置框架的
[`CLLocationCoordinate2D`](https://developer.apple.com/reference/corelocation/cllocationcoordinate2d)
类型来表示地理坐标。初始化
`CLLocationCoordinate2D`时，始终在经度之前指定纬度。

```objc
@import CoreLocation;

CLLocationCoordinate2D coord = CLLocationCoordinate2DMake(38.9099711, -77.0361122);
```

```swift
import CoreLocation

let coord = CLLocationCoordinate2D(latitude: 38.9099711, longitude: -77.0361122)
```
