## Optimization

Mapbox Optimization API 在输入坐标点之间，返回一个连续优化的路径。这也被称为[Traveling Salesperson Problem](https://en.wikipedia.org/wiki/Travelling_salesman_problem).Optimization API 的一个经典应用是在城市中规划配送路线。你可以为驾车，骑自行车或者步行获得一条路线。

**约束和限制**

- 最大 12 坐标点一次请求
- 最大 25 配送物一次请求
- 最大 60 请求每分钟

需要升级, [联系我们](https://www.mapbox.com/contact/).

### 检索优化

```endpoint
GET optimized-trips/v1/{profile}/{coordinates}
```

调用这个端点返回一条在输入坐标点之间，连续优化的路径。

URL parameter | Description
--- | ---
`profile` | Mapbox Directions 配置文件 ID. <table><tr><th>Profile ID</th><th>Description</th></tr><tr><td>`mapbox/driving`</td><td>汽车行驶时间、距离或两者。</td></tr><tr><td>`mapbox/walking`</td><td>步行和徒步旅行时间、距离或两者</td></tr><tr><td>`mapbox/cycling`</td><td>骑自行车时间、距离或两者</td></tr></table> 这个 Optimization API 不支持 `mapbox/driving-traffic` 配置文件。
`coordinates` | 一组用分号分隔的坐标序列`{longitude},{latitude}`。坐标点数必须在2到12之间. 第一个坐标点是路径规划的起点和终点。

你可以根据以下参数进一步优化来自该端点的结果：

Query parameter | Description
--- | ---
`annotations`<br /> (optional) | 返回路线附加数据。你可以包含一些注释，用逗号分隔的列表。接受: <table><tr><th>`annotations`</th><th>Description</th></tr><tr><td>`duration`</td><td>每组坐标之前的持续时间（秒）</td></tr><tr><td>`distance`</td><td>每组坐标之间的距离（米）</td></tr><tr><td>`speed`</td><td>每组坐标之间的速度（米/秒）</td></tr></table>
`approaches`<br /> (optional) | 一个逗号分隔的列表，表示在所求的道路中进入路标点的一侧。 包含 `unrestricted` (默认的，路线可以到达道路两边的路标点) 或者 `curb` (路线将到达在 `driving_side` 区域的路标点). 如果提供，approaches 的数量必须和路标点数量一致。 但是，你可以跳过坐标并用`;`分隔符显示其在列表中的位置。b必须和`steps=true`一起使用。
`bearings`<br /> (optional) | 影响路径从*starts* 开始的方向。 用于过滤道路中路标点被方向替换的部分。这有助于车辆在重新路径规划时能够继续其在当前方向行驶。一次这样的请求将提供第一个路标点的方向和半径值，其余值为空。必须与 `radiuses`参数一起使用。每个路标点取两个值：一个从真北顺时针方向在0到360之间的角度，以及角度可以偏离的范围 (建议值是 45° 或者 90°)，格式为`{angle, degrees}`.如果提供，bearings 的数量必须和路标点数量一致。 但是，你可以跳过坐标并用`;`分隔符显示其在列表中的位置。
`destination`<br />  (optional) | 指定返回路线的终点坐标。 接受 `any` (默认) 或者 `last`.
`distributions`<br /> (optional) | 通过提供一对与`coordinates`列表对应的，用`;`分隔列表，指定车辆行驶中的上车和下车地点。 第一个数字表示上车坐标对应的坐标序列下标， 第二个数字表示下车坐标对应的坐标序列下标。 每对必须精确包含两个数字，不能是相同的。返回的结果必须在下车地点之前经过上车地点。第一个坐标只能是上车地点，不能是下车地点。
`geometries`<br /> (optional) | 返回geometry的格式。 允许的值包括: `geojson` (as [LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2)), [`polyline`](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) (default, a polyline with precision 5), [`polyline6`](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) (a polyline with precision 6).
`language`<br /> (optional) | 返回的语言是文本语言. 详见 [supported languages](#instructions-languages)。 默认是 `en` (English).
`overview`<br /> (optional) | 返回的整体几何结构类型。可以是 `full` (最详细的几何信息), `simplified` (默认，完整几何的简化版本), or `false` (没有概述几何).
`radiuses`<br /> (optional) | 坐标可以被移动到道路网络中的最大距离（米）。请求中必须有多个半径值，用`;`分隔。值可以是任意大于`0`或者字符串`unlimited`. 如果在半径范围内找不到可行驶的路径，返回一个`NoSegment`错误。
`source`<br />  (optional) | 开始返回路径的坐标。 接受 `any` (默认) or `first`.
`steps`<br /> (optional) | 是否返回步骤和转向信息 (`true`) 或者不是 (`false`, 默认)。
`roundtrip`<br />  (optional) | 表示返回的路径是否是一个返回行程，意味着路径返回到第一个坐标点(`true`, 默认) 或者不是 (`false`). 如果`roundtrip=false`, 参数 `source` 和 `destination` 是需要的，但是不是所有的组合都是可能的。 见 *Fixing Start and End Points* 部分获取更多信息。

查询字符串中包含未识别的选项会导致 `InvalidInput` 错误。

注意，如果 Optimization API中的每一个点都设置[`continue_straight=false`](#retrieve-directions)，意味着返回路径将沿着相同方向。 参见 [`continue_straight`](#retrieve-directions)Directions API 中的参数，了解这条路径的更多信息。

**固定起点和终点**

可以显式地设置行程的开始或结束坐标：

- 当 `source=first`, 在输出中使用第一个坐标作为路径的起始坐标。
- 当 `destination=last`， 在输出中使用最后一个坐标作为路径的终点坐标。
- 如果你指定 `any` 为 `source` 或者 `destination`, 任何坐标都可以用作输出中的第一个或最后一个坐标。
- 如果 `source=any&destination=any`, 返回的往返路径将在默认情况下在第一个输入坐标处开始。

不是所有的 `roundtrip`, `source`, 和 `destination` 的组合是支持的. 现在，下面的组合是可行的：

| roundtrip | source | destination | supported |
| :-- | :-- | :-- | :-- |
| true | first | last | **yes** |
| true | first | any | **yes** |
| true | any | last | **yes** |
| true | any | any | **yes** |
| false | first | last | **yes** |
| false | first | any | no |
| false | any | last | no |
| false | any | any | no |

#### 请求举例

```curl
# 请求无额外选项的最佳车程
curl "https://api.mapbox.com/optimized-trips/v1/mapbox/driving/-122.42,37.78;-122.45,37.91;-122.48,37.73?access_token={your_access_token}"

# 利用steps和GeoJSON响应请求最佳自行车车程
curl "https://api.mapbox.com/optimized-trips/v1/mapbox/cycling/-122.42,37.78;-122.45,37.91;-122.48,37.73?steps=true&geometries=geojson&access_token={your_access_token}"

# 请求在柏林用四个坐标优化汽车往返行程，从第一个坐标对开始，到最后一个坐标结束
curl "https://api.mapbox.com/optimized-trips/v1/mapbox/driving/13.388860,52.517037;13.397634,52.529407;13.428555,52.523219;13.418555,52.523215?source=first&destination=last&roundtrip=true&access_token={your_access_token}"

# 请求具有四个坐标和一个分布约束的优化汽车行程，其中必须在第二个坐标之前访问最后一个给定坐标
curl "https://api.mapbox.com/optimized-trips/v1/mapbox/driving/13.388860,52.517037;13.397634,52.529407;13.428555,52.523219;13.418555,52.523215?roundtrip=true&distributions=3,1&access_token={your_access_token}"

# 请求优化的车程，通过指定路标点顺序和方向
curl "https://api.mapbox.com/optimized-trips/v1/mapbox/driving/-122.42,37.78;-122.45,37.91;-122.48,37.73?radiuses=unlimited;unlimited;unlimited&bearings=45,90;90,1;340,45&steps=true&access_token={your_access_token}"
```

```python
# 这个API是不可用的通过Python SDK
```

```javascript
// 这个API是不可用的通过JavaScript SDK
```

```bash
# 这个API是不可用的通过Mapbox CLI
```

```java
MapboxOptimization optimizedClient = MapboxOptimization.builder()
  .source(DirectionsCriteria.SOURCE_FIRST)
  .destination(DirectionsCriteria.DESTINATION_ANY)
  .coordinates(listOfPoints)
  .overview(DirectionsCriteria.OVERVIEW_FULL)
  .profile(DirectionsCriteria.PROFILE_DRIVING)
  .accessToken("{your_access_token}")
  .build();
```

```objc
// 这个APIT无法访问通过Mapbox Objective-C libraries
```

```swift
// 这个APIT无法访问通过Mapbox Swift libraries
```

#### 返回举例

```json
{
    "code": "Ok",
    "waypoints": [
        {
            "name": "North Lake Boulevard",
            "location": [ -120.141159, 39.170872 ],
            "waypoint_index": 0,
            "trips_index": 0
        },
        {
            "name": "Virginia Drive",
            "location": [ -120.14984, 39.159985 ],
            "waypoint_index": 2,
            "trips_index": 0
        },
        {
            "name": "Fairway Drive",
            "location": [ -120.150648, 39.340689 ],
            "waypoint_index": 1,
            "trips_index": 0
        }
    ],
    "trips": [
        {
            "geometry": "}panFfah|Ujj@ru@`Dp`BwNdpAwc@pw@ibAxm@snA|Ic^|\\q{@{S}`@lVewBzUa}@t^}oAdEkbB}[{{AvHqdDs\\qn@lV_OqpA{`A}bBaRucA_gB}eCbI_MzXzY`Va]xUbPzE|[{E}[_WoP{Tl]{X{YoLfQtjBvdCbRpcA|_A|`BbD`qA~y@{WpdDr\\z{AwHh}A`\\hsA{Dv~@c_@dwB{U|`@mVp{@zSfe@_`@niA}G|aAin@~b@mx@|LunBsBw}@{Sia@bp@nDcCpYbCqYou@uDsP_U",
            "legs": [
                {
                    "summary": "",
                    "weight": 1962.8,
                    "duration": 1876.9,
                    "steps": [],
                    "distance": 31507.9
                },
                {
                    "summary": "",
                    "weight": 2211.9,
                    "duration": 2035.1,
                    "steps": [],
                    "distance": 32720.5
                },
                {
                    "summary": "",
                    "weight": 283.5,
                    "duration": 238.1,
                    "steps": [],
                    "distance": 1885.2
                }
            ],
            "weight_name": "routability",
            "weight": 4458.2,
            "duration": 4150.1,
            "distance": 66113.6
        }
    ]
}
```

### Optimization response object
对 Optimization API请求的响应是包含下列属性的JSON对象：

Property | Description
--- | ---
`code` | 指示响应状态的字符串。这是一个独立于HTTP状态的代码。 在正常有效响应上，该值将为 `Ok`。
`waypoints` | 一个数组 `waypoint` 每个路标点是一个到道路和路径网络的输入坐标。路标点以输入坐标的顺序出现在数组中。
`trips` | 一组0 或者1 `trip` 数组对象。

**Waypoint objects**

一个 **waypoint object** 是到道路网络的输入坐标，包含以下特征：

Property | Description
--- | ---
`name` | 一个字符串，表示输入坐标连接的道路名称。
`location` | 一个数组包含`[longitude, latitude]` 连接的坐标点。
`trips_index` | 这个`trip` 对象的下标，包含路标点在 `trips` 数组中的位置。
`waypoint_index` | 给定路标点在`trip`中的位置索引。

**Trip object**

一个 **trip object** 描述通过多个路标点的路径，并具有以下属性： 

Property | Description
--- | ---
`geometry` | 根据 `geometries` 参数， 这是一个 [GeoJSON LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2) 或者 [Polyline string](https://developers.google.com/maps/documentation/utilities/polylinealgorithm). 根据 `overview` 参数， 这是完整的路线几何结构 (`full`), 一种简化的几何结构，用于缩放可以完全显示路线 (`simplified`)，或者不包含(`false`).
`legs` | 一个数组 [route leg](#routeleg-object) 对象.
`weight_name` | 一个字符串表示使用哪个权重。默认的是 `routability`, 这是基于时间的，对于不太理想的策略添加惩罚。
`weight` | 一个浮点数表示单位权重，通过 `weight_name`.
`duration` | 一个浮点数表示估计的行程时间（秒）。
`distance` | 一个浮点数表示行程的距离（米）。

一个行程对象有与[route object](#route-object)相同的格式在Directions API中。

#### Example response object

```json
{
    "code": "Ok",
    "waypoints": [
        {
            "location": [ -6.80897, 62.000075 ],
            "waypoint_index": 0,
            "name": "Kirkjubøarvegur",
            "trips_index": 0
        },
        {
            "location": [ -6.802374, 62.004142 ],
            "waypoint_index": 1,
            "name": "Marknagilsvegur",
            "trips_index": 0
        }
    ],
    "trips": [
        {
            "distance": 1660.8,
            "duration": 153,
            "legs": [
                {
                    "summary": "",
                    "duration": 77.3,
                    "steps": [],
                    "distance": 830.4
                },
                {
                    "distance": 830.4,
                    "steps": [],
                    "duration": 75.7,
                    "summary": ""
                }
            ],
            "geometry": "oklyJ`{ph@yBuY_F{^_FxJoBrBs@d@mATlAUr@e@nBsB~EyJ~Ez^xBtY"
        }
    ]
}
```

### Optimization errors

在错误时，服务器用不同的HTTP状态代码进行响应。对于HTTP状态码低于`500`的响应，JSON 响应主体包含 `code` 属性，客户端程序可以使用它来管理控制流。响应体还可以包括 `message` 属性，并对错误进行可读的解释。

如果发生服务器错误，HTTP状态代码将为`500`或更高，响应将不包括`code` 属性。

响应体 `code` | HTTP 状态代码| Description
--- | --- |---
`Ok` | `200` | 正常成功案例。
`NoTrips` | `200` | 对于一个坐标，找不到到其他坐标的路径。检查不可能的路线（例如，在没有渡轮连接的海洋上）。
`NotImplemented`  | `200` | 对于给定的组合`source`, `destination`, 和 `roundtrip`, 这个请求是不支持的。
`ProfileNotFound` | `404` | 使用一个无效的配置文件作为描述 [Retrieve an optimization](#retrieve-an-optimization).
`InvalidInput` | `422` | 给定的请求是无效的。响应的`message`键将保持对无效输入的解释。

其他的属性可能未定义。
