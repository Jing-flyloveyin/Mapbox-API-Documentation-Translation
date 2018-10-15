## 阅读这篇文档

这篇文档由如[地理编码](#geocoding)或[上传](#uploads)的一组相关功能的API和端点组成，这些端点是在API内执行某个操作并位于特定URL的特定方法。

本文档的每个端点都可分成以下几个部分进行描述：

* HTTP方法：包括GET，POST，PUT，PATCH和DELETE
* 路径：例如，`/geocoding/v5/{mode}/{query}.json`
* URL参数：这些参数是端点路径中用括号括起来的那部分，像本例中的 `{mode}` 。
* 查询参数：包含在带有 _Option_ 请求头的表中，这些参数被添加到请求的查询字符串中。
* 令牌（token）作用域，如果需要访问令牌。

本文档中所有引用的URL都有一个基本路径 `https://api.mapbox.com`。
该基本路径位于端点路径的 _前面_。在本例中，你可以将 `https://api.mapbox.com` 和 `/geocoding/v5/{mode}/{query}.json` 合并，得到URL `https://api.mapbox.com/geocoding/v5/{mode}/{query}.json` 。

对于该端点，`{mode}` 和 `{query}` 是URL参数。在请求中，需要把对应的占位符替换成真值。例如，选择 `mapbox.places` 作为你的mode以及 `Chester` 作为你的query，从而得到URL `https://api.mapbox.com/geocoding/v5/mapbox.places/Chester.json` 。

查询参数是使用[查询字符串编码](https://en.wikipedia.org/wiki/Query_string)添加到URL的结尾。如果想要添加 `country` 这个查询参数到地理编码请求中，你可以将查询字符串 `?country=us` 添加到URL的结尾，得到 `https://api.mapbox.com/geocoding/v5/mapbox.places/Chester.json?country=us`。

所有端点都需要访问令牌，该令牌作为查询参数提供。所以最后你组成的地理编码请求将会像这样
`https://api.mapbox.com/geocoding/v5/mapbox.places/Chester.json?country=us&access_token=pk.my-token-value` 。
下一节将介绍如何获取以及使用令牌。

```http
https://api.mapbox.com
```

## 访问令牌

访问Mapbox的Web服务时，需要一个[访问令牌](https://mapbox.com/help/define-access-token)，该访问令牌会连接API访问到你的账户。本文档中的请求示例都不包含访问令牌。你将需要通过 `access_token` 查询选项来提供一个令牌，或者在SDK或库中指定一个。

你的默认访问令牌可以在你的[账户面板](https://www.mapbox.com/account)获取。你还可以在你的[令牌页面](https://www.mapbox.com/account/access-tokens/)或者通过[Tokens API](#tokens)，来管理并创建更多的访问令牌。

#### 访问令牌例子

```http
https://api.mapbox.com/{endpoint}?access_token={your_access_token}
```

当创建一个新的访问令牌时，你将有一个选项，可以添加一个或以上的**作用域**。每一个作用域都为访问令牌添加不同权限，允许其用于访问受限制的API。在整个文档中，我们指定了访问每个端点所需的作用域。

## 版本控制

每个 Mapbox API 都使用基本URL中指定的版本字符串进行版本控制，该版本字符串可以独立于其他API进行修改。

#### 版本控制例子

```http
https://api.mapbox.com/{api}/{version}
```

_[Maps API](#maps) 是一个例外：他的端点是以 `/v4/{map_id}` 为前缀，而不是将版本放在API名称之后。该特例将在下一个API版本修复。_

我们始终鼓励使用最新可用的API。

以下这些更改被认为是向后兼容的，并且不需要更新版本字符串：

- 向JSON对象增加一些属性。
- 更改单个列表请求返回的条目数。
- 更改速率限制的阈值。
- 通过API生成的标识符的结构或长度。
- 更改错误信息。

这些更改被认为是不向后兼容的，并且需要更新版本字符串：

- 向JSON对象移除一些属性。
- 更改API的URL结构。

如果某些功能已弃用，我们将至少提前90天通过电子邮件通知你。仅当我们检测到你正在使用部分已弃用的API，你才会收到一封弃用告警电子邮件。

## 速率限制

Mapbox API 具有速率限制，可以限制对端点发出的请求数。 如果超出速率限制，你的请求将受到限制，并且将接收到来自API的响应 `HTTP 429 Too Many Requests` 。

| Header | Description |
| --- | --- |
| `X-Rate-Limit-Interval` | 速率限制的时间间隔（以秒为单位） |
| `X-Rate-Limit-Limit` | 在当前时间间隔内最大的请求数|
| `X-Rate-Limit-Reset` | 当前时间间隔将结束以及速率限制的计数重置的时间（Unix时间戳）|

## 跨域资源共享（CORS）

Mapbox的web服务支持[跨域请求](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)，不受域限制。为了支持IE8和IE9，请使用回退到XDomainRequest的库，例如[corslite](https://github.com/mapbox/corslite/)。

## Retina

```http
# 一张像素400x200的静态地图
https://api.mapbox.com/v4/mapbox.dark/-76.9,38.9,5/400x200.png?access_token={your_access_token}

# Retina显示的相同静态地图：
# 这个图像像素将是800x400，
# 但显示一样的内容
# 并且缩小时看起来是一样的
https://api.mapbox.com/v4/mapbox.dark/-76.9,38.9,5/400x200@2x.png?access_token={your_access_token}
```

Mapbox支持所有提供图像服务的API上的Retina图像输出。在URL上的文件扩展名之前添加 `@2x` 来请求两倍比例的图像。例如，使用 `@2x` 将像素256×256的地图瓦片将变成像素512×512，但显示的图像内容是一样的。当在页面上显示时，图像的大小仍然是256×256像素，但是原始图像的4个像素将以屏幕单位表现为1个像素。

URL上的 `@2x` 部分优先级高于整个格式，因此是以 `.png` 结尾的URL将以`@2x.png`作为请求Retina图像的结尾。

唯一不能使用Retina缩放的资源是由TileMill上传的或者MBTiles格式的瓦片集。

## HTTPS

我们建议通过HTTPS访问Mapbox。除了 Maps API，通过HTTP发起的请求都会自动升级为HTTPS。

## 分页

```curl
Link: <https://api.mapbox.com/uploads/v1/1454024795582?start=cijywvxlm004rtikohhqf99jv&limit=100>; rel="next"
```

分页允许你通过多次请求获取多个对象列。接收到了一页对象后，可以通过在这一页请求响应的 [`Link` header](http://tools.ietf.org/html/rfc5988) 中的 `next` 链接关系来请求下一页。这个过程可以不断重复，直到服务器发回一个没有 `Link` header 或者没有 `next` 链接关系的响应，该响应意味着对象集合已全部请求完毕。

你的应用*必须*使用 `Link` header 进行分页，而不是
构建自己的URL，因为用于分页的特定URL可能会随时更改。[Python requests library](http://docs.python-requests.org/en/master/user/advanced/#link-headers)
和 [link-header-parser module for JavaScript](https://github.com/thlorenz/parse-link-header) 都可以解析 Link header。Link header 遵循 [RFC 5988 specifications](http://tools.ietf.org/html/rfc5988)。

[数据集](#datasets)、[上传](#uploads)、[样式](#styles)、[瓦片集](#tilesets)和[令牌](#tokens)端点列表都支持分页。

| Query Parameter | Description |
| --- | --- |
| `limit` | 每一页返回的*最大*对象数。API将尝试返回所请求数量的对象，但接收到数量较少的对象时并不一定表示对象集合接收已结束。接收到没有 `Link` header 或没有`next` 链接关系的响应是确定何时集合接收结束的唯一方法。

## 日期

API返回的大多数日期和时间是 [RFC 3339](https://tools.ietf.org/html/rfc3339) 格式。[JavaScript Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) constructor，[Python arrow library](https://github.com/crsmithdev/arrow)，以及其他库与开发语言，都可以解析该格式。

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

这个规则的唯一例外是 [Retrieve TileJSON metadata](＃retrieve-tilejson-metadata)
端点，其返回的 `created` 和 `modified` 属性是 [Unix time](https://en.wikipedia.org/wiki/Unix_time) 格式。

```objc
@import Foundation;

NSDate *date = [NSDate dateWithTimeIntervalSince1970:1416598870000];
```

```swift
import Foundation

let date = Date(timeIntervalSince1970: 1_416_598_870_000)
```

## 坐标系

如果地理坐标需要用于 Mapbox API，那么他们应按照 `经度，纬度` 顺序进行格式化，并在WGS84坐标系中必须为十进制的度数。该模式符合现有标准，包括GeoJSON和KML。

Mapbox API 尽可能使用GeoJSON格式来表达地理空间数据。方向、地图匹配、地理编码和数据集等API的请求响应都返回GeoJSON格式，上传和地图匹配等API都支持传入GeoJSON。

`经度，纬度` 顺序的唯一例外是在静态（经典）叠加和方向响应中支持的polyline格式。当polyline指定输入或输出时，polyline内容应遵循 Google Encoded Polyline 格式，即 `纬度，经度` 顺序。

Mapbox Swift 库使用核心位置框架 [`CLLocationCoordinate2D`](https://developer.apple.com/reference/corelocation/cllocationcoordinate2d) 用于表达地理坐标。初始化 `CLLocationCoordinate2D` 时，始终指定纬度在经度之前。

```objc
@import CoreLocation;

CLLocationCoordinate2D coord = CLLocationCoordinate2DMake(38.9099711, -77.0361122);
```

```swift
import CoreLocation

let coord = CLLocationCoordinate2D(latitude: 38.9099711, longitude: -77.0361122)
```
