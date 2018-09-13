## Reading this Documentation

This documentation is structured by API, which is a group of related functionality
like [Geocoding](#geocoding) or [Uploads](#uploads), and then by endpoint, which
is a specific method within that API that performs one action and is located
at a specific URL.

Each endpoint in this documentation is described using several parts:

* The HTTP method: includes GET, POST, PUT, PATCH, DELETE
* The path: for instance, `/geocoding/v5/{mode}/{query}.json`
* URL parameters: these are the parts of the endpoint path wrapped in brackets,
  like `{mode}` in this example.
* Query parameters: contained in a table with an _Option_ header, these are added
  to the query string part of the request.
* A token scope, if one is required.

All URLs referenced in the documentation have the base path `https://api.mapbox.com`.
This base path goes _before_ the endpoint path. In this example, you'd
combine `https://api.mapbox.com` and `/geocoding/v5/{mode}/{query}.json` to get
the request URL `https://api.mapbox.com/geocoding/v5/{mode}/{query}.json`.

For this endpoint, `{mode}` and `{query}` are the URL parameters. In a request,
you replace their placeholders with real values: for instance, you'd choose
`mapbox.places` as your mode and `Chester` as your query, and get the URL
`https://api.mapbox.com/geocoding/v5/mapbox.places/Chester.json`

Query parameters are added to the end of the URL with [query string encoding](https://en.wikipedia.org/wiki/Query_string).
If you wanted to add the `country` query parameter to that Geocoding request, you'd
the query string `?country=us` to the end of the URL, producing
`https://api.mapbox.com/geocoding/v5/mapbox.places/Chester.json?country=us`.

All endpoints require an access token, which is provided as a query parameter.
So the final geocoding request you would construct would look like
`https://api.mapbox.com/geocoding/v5/mapbox.places/Chester.json?country=us&access_token=pk.my-token-value`
The next section covers how you get and use access tokens.

```http
https://api.mapbox.com
```

## Access tokens

Access to Mapbox web services requires an [access token](https://mapbox.com/help/define-access-token) that connects API
requests to your account. The example requests in this documentation don't include
an access token: you will need to supply one using the `access_token` query
option or by specifying the token in the SDK or library.

Your default access token is available on your
[Account Dashboard](https://www.mapbox.com/account). You can also manage
and create additional tokens on your [Access tokens page](https://www.mapbox.com/account/access-tokens/) or with the [Tokens API](#tokens).

#### Access token example

```http
https://api.mapbox.com/{endpoint}?access_token={your_access_token}
```

When creating a new access token, you have the option of adding one or more **scopes**.
Each scope adds a different permission to the token, allowing it to be used to
access restricted APIs. Throughout the documentation, we specify the scope
required to access each endpoint.

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
# a 400x200 static map
https://api.mapbox.com/v4/mapbox.dark/-76.9,38.9,5/400x200.png?access_token={your_access_token}

# the same static map for Retina
# displays: this image will be 800x400
# pixels, but show the same content
# and look the same when scaled down
https://api.mapbox.com/v4/mapbox.dark/-76.9,38.9,5/400x200@2x.png?access_token={your_access_token}
```

Mapbox supports Retina image output on all APIs that serve images.
Add `@2x` before the file extension on a URL to request an image at
double scale. For instance, a map tile that is 256×256 pixels will be
512×512 pixels with `@2x`, but show the same content. When displayed
on a page, the image will be still sized to 256×256 pixels, but
4 pixels of the original will represent 1 pixel in screen units.

The `@2x` part of the URL goes before the entire format, so a URL
that ends in `.png` would end with `@2x.png` as a Retina image.

The only assets that are not available at Retina scale are tilesets
uploaded from TileMill or as MBTiles.

## HTTPS

We recommend all access to Mapbox is over HTTPS. Except for the
Maps API, requests initiated over HTTP are automatically upgraded to HTTPS.

## Pagination

```curl
Link: <https://api.mapbox.com/uploads/v1/1454024795582?start=cijywvxlm004rtikohhqf99jv&limit=100>; rel="next"
```

Pagination lets you list many objects from an API by using more than one
request. After receiving a page of objects, the next page can be requested using
the `next` link relation in [`Link` header](http://tools.ietf.org/html/rfc5988)
of the response. This process can be repeated until the server sends a response
without a `Link` header or without a `next` link relation, which signals the end
of the collection.

Your application *must* use the `Link` header for pagination instead of
constructing your own URLs as the specific URLs used for pagination may change
at any time. The
[Python requests library](http://docs.python-requests.org/en/master/user/advanced/#link-headers),
and the
[link-header-parser module for JavaScript](https://github.com/thlorenz/parse-link-header)
can parse Link headers. Link headers follow the [RFC 5988 specifications](http://tools.ietf.org/html/rfc5988).

Pagination is supported on the [Datasets](#datasets), [Uploads](#uploads), [Styles](#styles), [Tilesets](#tilesets), and [Tokens](#tokens) list endpoints.

| Query Parameter | Description |
| --- | --- |
| `limit` | The *maximum* number of objects to return. The API will attempt to return the requested number of objects, but receiving fewer objects does not necessarily signal the end of the collection. Receiving a response with no `Link` header or no `next` link relation is the only way to determine when you are at the end of a collection. |

## Dates

Most dates and times returned by the API are represented
in [RFC 3339](https://tools.ietf.org/html/rfc3339) format, which can be
parsed by the [JavaScript Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)
constructor, the [Python arrow library](https://github.com/crsmithdev/arrow),
and many other libraries and languages.

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

The only exception to this rule is the [Retrieve TileJSON metadata](#retrieve-tilejson-metadata)
endpoint that returns `created` and `modified` properties as [Unix time](https://en.wikipedia.org/wiki/Unix_time).

```objc
@import Foundation;

NSDate *date = [NSDate dateWithTimeIntervalSince1970:1416598870000];
```

```swift
import Foundation

let date = Date(timeIntervalSince1970: 1_416_598_870_000)
```

## Coordinates

Where geographic coordinates are provided to a Mapbox API, they should be formatted
in the order `longitude, latitude` and specified as decimal degrees in the WGS84
coordinate system. This pattern matches existing standards, including GeoJSON and KML.

Mapbox APIs use GeoJSON formatting wherever possible to represent
geospatial data. The Directions, Map Matching, Geocoding, and Datasets APIs
all return GeoJSON-formatted responses, and the Upload and Map Matching APIs
accept GeoJSON input.

The only exception to `longitude, latitude` ordering is the polyline format, supported
in Static (Classic) overlays and Directions responses. When polyline input
or output is specified, the polyline content should follow the Google Encoded Polyline format,
which specifies `latitude, longitude` ordering.

The Mapbox Swift libraries use the Core Location framework’s
[`CLLocationCoordinate2D`](https://developer.apple.com/reference/corelocation/cllocationcoordinate2d)
type to represent geographic coordinates. When initializing a
`CLLocationCoordinate2D`, always specify the latitude before the longitude.

```objc
@import CoreLocation;

CLLocationCoordinate2D coord = CLLocationCoordinate2DMake(38.9099711, -77.0361122);
```

```swift
import CoreLocation

let coord = CLLocationCoordinate2D(latitude: 38.9099711, longitude: -77.0361122)
```
