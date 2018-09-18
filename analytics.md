## Analytics

<!-- preview -->

*这份API适用于 [ Commercial and Enterprise ](https://www.mapbox.com/pricing/) 计划。*

此Mapbox Analytics API 将按资源返回服务的API使用情况。例如，它可以通过一个特定的访问token来计算过去一周内发起的地理编码请求的次数。
```python
from mapbox import Analytics
```

### Retrieve analytics

根据给出的资源和时段返回按天统计的某一请求发起的次数。

* 如果 `{resourceType}` 是 `tokens`, 则 `{id}` 是完整的token。
* 如果 `{resourceType}` 是 `styles`, 则 `{id}` 是Style ID。注意不要与 Style _URL_ 混淆，此 id 是样式末尾的字母数字部分： 此 Style URL `mapbox://styles/user/cimdoca6f00` 包含Style ID `cimdoca6f00`， 所以此分析请求的路径则为 `/analytics/v1/styles/user/cimdoca6f00`。
* 如果 `{resourceType}` 是 `tilesets`, 则 `{id}` 是一个 Map ID.

```endpoint
GET /analytics/v1/{resourceType}/{username}/{id}?period={period} analytics:read
```

URL 参数 | 描述
--- | ---
`resourceType` | 被请求的资源类型。可用的类型有 `accounts`， `tokens`， `tilesets` 和 `styles`。
`username` | 拥有此资源的账户用户名。
`id` <br /> (可选) | 此资源的id。如果类型为 `accounts` 则此项不必需。
`period` <br /> (可选) | 由两个逗号分割的格式为 [ISO formatted strings](#dates)的日期。 第一个日期必须早于第二个日期。区间包括给出的日期。如果未提供此项，则默认为过去的90天。最长区间为1年。如果提供的两个日期相隔超过1年，则会返回一个错误。


#### Example Request

```curl
curl "https://api.mapbox.com/analytics/v1/tilesets/mapbox/mapbox.streets?period=2016-03-22T00:00:00.000Z,2016-03-24T00:00:00.000Z&access_token={your_access_token}"
```

```bash
# This API cannot be accessed with Mapbox CLI
```

```javascript
// This API cannot be accessed with the JavaScript SDK
```

```python
analytics = Analytics()
response = analytics.analytics('accounts', '{username}')
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
    "period": ["2016-03-22T00:00:00.000Z", "2016-03-24T00:00:00.000Z"],
    "timestamps": ["2016-03-22T00:00:00.000Z", "2016-03-23T00:00:00.000Z", "2016-03-24T00:00:00.000Z"],
    "services": {
        "mapview": [ 25, 22, 37 ],
        "static": [ 23, 20, 34 ],
        "tiles": [ 30, 39, 53 ]
    }
}
```

包含每项服务各请求数目数组的响应。

属性 | 描述
--- | ---
`timestamps` | 包含在此响应中的 ISO formatted strings 类型的日期数组。
`period` | 一个包含两个元素的数组，分别为此响应区间的 ISO formatted strings 类型的开始与结束日期。
`services` | 每一项服务对应的一个带密钥对象。其值为记录每天请求数量的一个数组，顺序与 `timestamps` 中的时序一致。

只有适用于给出的资源的服务会在响应中被返回。

资源类型 | 在响应中被返回的服务
--- | ---
`accounts` | `mapview` `static` `tile` `directions` `geocode`
`tokens` | `mapview` `static` `tile` `directions` `geocode`
`tilesets` | `mapview` `static` `tile`
`styles` | `mapview` `static` `tile`

对于 `styles` 资源类型， Static 和 Tile 服务请参考 [Static API](#static)。 对于 `tilesets`资源类型， Static 和 Tile 服务请参考 [Static (Classic)](#static-classic) 和 [Maps → Tiles](#retrieve-tiles) APIs。
