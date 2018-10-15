## Analytics

<!-- preview -->

*此API适用于 [商业与企业](https://www.mapbox.com/pricing/) 计划。*

Mapbox Analytics API 可以根据资源类型返回服务API的使用情况。例如它可以计算出一周内带有特殊token的地理编码服务的请求次数。

```python
from mapbox import Analytics
```
### 检索分析

Returns the request counts per day for given resource and period.
返回指定资源或时期的每日请求次数。

* 如果 `{resourceType}` 是 `tokens`，则`{id}`为完整的token。

* 如果`{resourceType}` 是 `styles`，则 `{id}` 为样式ID。不要与样式URL混淆，样式URL中id 为尾部的字母数字序列：例如样式URL `mapbox://styles/user/cimdoca6f00` 包含样式ID  `cimdoca6f00` ，所以分析请求的路径为  `/analytics/v1/styles/user/cimdoca6f00`。
* 如果`{resourceType}` 是 `tilesets`，则  `{id}` 为地图ID。

```endpoint
GET /analytics/v1/{resourceType}/{username}/{id}?period={period} analytics:read
```

URL 参数 | 描述
--- | ---
`resourceType` | 被请求的资源类型。有效的资源类型为 `accounts`, `tokens`, `tilesets` 和 `styles`.
`username` | 资源所有者账户的用户名。
`id` <br /> (可选) | 资源ID。 当资源类型为 `accounts` 时此选项非必须。
`period` <br /> (可选) | 两个使用逗号分隔的[ISO 格式字符串](#dates)日期。第一个日期必须早于第二个。时期字段由此日期段指定，默认为90天。最长时期为1年，如果指定的日期超过1年则返回错误。

#### 请求示例

```curl
curl "https://api.mapbox.com/analytics/v1/tilesets/mapbox/mapbox.streets?period=2016-03-22T00:00:00.000Z,2016-03-24T00:00:00.000Z&access_token={your_access_token}"
```

```bash
# 此API不能使用 Mapbox CLI 访问
```

```javascript
// 此API不能使用 JavaScript SDK 访问
```

```python
analytics = Analytics()
response = analytics.analytics('accounts', '{username}')
```

```java
// 此API不能使用 Mapbox Java SDK 访问。
```

```objc
// 此API不能使用 Mapbox Objective-C 库访问。
```

```swift
// 此API不能使用 Mapbox Swift 库访问
```

#### 响应示例


```json
{
    "period": ["2016-03-22T00:00:00.000Z", "2016-03-24T00:00:00.000Z"],
    "timestamps": ["2016-03-22T00:00:00.000Z", "2016-03-23T00:00:00.000Z", "2016-03-24T00:00:00.000Z"],
    "services": {
        "mapview": [ 25, 22, 37 ],
        "static": [ 23, 20, 34 ],
        "tiles": [ 30, 39, 53 ]
    }
}
```

响应包含每个服务请求次数的数组。

属性 | 描述
--- | ---
`timestamps` | 响应中所包含的每一天的ISO格式的日期字符串数组。
`period` | 响应时期中的起止日期的ISO格式字符串数组。
`services` | 包含每种服务次数的对象。键值为服务名，值为与 `timestamps` 中的日期相对应的每日请求次数。

响应中资源类型与所返回的服务。

资源类型 | 所返回的服务
--- | ---
`accounts` | `mapview` `static` `tile` `directions` `geocode`
`tokens` | `mapview` `static` `tile` `directions` `geocode`
`tilesets` | `mapview` `static` `tile`
`styles` | `mapview` `static` `tile`


对于 `styles` 资源类型，Static & Tile 服务请参考[Static API](#static)。对于`tilesets`资源类型，Static & Tile 服务请参考 [Static (Classic)](#static-classic) 和 [Maps → Tiles](#retrieve-tiles) API。
