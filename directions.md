## 路径规划

本文档是Directions API的`v5`版。之前版本请参考[`v4`文档](./pages/directions-v4.html).

Mapbox Directions API 将向你展示如何到达目的地。通过使用Directions API，你可以:

- 计算驾车、步行、骑行的路况感知与事件感知的最佳路径
- 生成多段导航命令
- 支持25个地球上任意坐标对生成路径

该API支持以下四种不同的路线规划配置文件：

配置文件 | 说明
--- | ---
`mapbox/driving-traffic` | 自动路线规划。考虑当前和历史的交通信息，规避拥堵。交通信息可参考：[these supported geographies](./pages/traffic-countries.html).
`mapbox/driving` | 自动路线规划。提供优先高速（如公路）的最快路线。
`mapbox/walking` | 行人和徒步。 提供人行道和小路的最短路径。
`mapbox/cycling` | 骑行路线。规避高速公路，优先选择带有自行车道的道路，对于骑行者而言路线更短且更安全。

Swift和Objective-C对Directions的支持可参考库：[MapboxDirections.swift](https://github.com/mapbox/MapboxDirections.swift)。

```objc
@import MapboxDirections;

MBDirections *directions = [[MBDirections alloc] initWithAccessToken:@"<#your access token#>"];
```

```objc
@import MapboxDirections;

// If you’ve already set `MGLMapboxAccessToken` in your Info.plist:
MBDirections *directions = [MBDirections sharedDirections];
```

```swift
import MapboxDirections

let directions = Directions(accessToken: "<#your access token#>")
```

```swift
import MapboxDirections

// If you’ve already set `MGLMapboxAccessToken` in your Info.plist:
let directions = Directions.shared
```

```python
from mapbox import Directions
```

```javascript
const mbxDirections = require('@mapbox/mapbox-sdk/services/directions');
const directionsClient = mbxDirections({ accessToken: '{your_access_token}' });
```

**限制**

- 使用配置文件`driving`，`walking`和 `cycling`的请求可沿路径指定最多25个点（输入坐标将捕捉路网）。
- 使用配置文件`driving-traffic`的请求可沿路径指定最多3个点。
- [supported geographies](./pages/traffic-countries.html) 提供了`driving-traffic`配置文件的交通信息。该配置文件在没有交通信息的地区则返回 `driving`文件。
- 每分钟最多支持60次请求。
- 所有点之间距离最多10000公里。

### 检索路径

```endpoint
GET /directions/v5/{profile}/{coordinates}
```

在路径点之间检索路径。路径请求必须明确至少两个路径点作为起点和终点。

可在[API Playground](https://www.mapbox.com/api-playground/#/directions/)中进行尝试。

URL 参数 | 说明
--- | ---
`profile` | 如果使用路径配置文件，可能的值有`mapbox/driving-traffic`，`mapbox/driving`，`mapbox/walking`或者`mapbox/cycling`。
`coordinates` | 坐标对`{longitude},{latitude}`列表用分号隔开，并按顺序访问。对于大多数请求，可以有2到25对坐标，对于`driving-traffic`请求，最多可以有3对坐标。

你可以使用下列可选参数进一步优化端点的结果：

查询参数 | 说明
--- | ---
`alternatives`<br /> （可选） | 是否尝试返回替代路线(`true`)或者不返回(`false`，默认值)。替代路线是与最快路线明显不同的路线，但仍然相当快。并不是所有情况下都存在该路线，最多可返回2个替代路线。
`annotations`<br /> （可选） | 沿路线返回其他元数据。可能的值有：`duration`，`distance`，`speed`和 `congestion`。多个注释列表可用逗号分隔。注释中包含的详细内容，请参考：[route leg object](#routeleg-object)。必须与`overview=full`一起使用。
`approaches`<br /> （可选） | 用分号分隔的列表指示的是请求路径中路径点应在道路的哪一侧。选择`unrestricted`（默认值，路径生成的路径点可以在道路的任意一侧）或者 `curb` (路径生成的路径点将沿着该地区`driving_side`的方向)。如果选择，方法的数量必须与路径点数量相同。但是，你可以跳过坐标并通过`;`分隔符表示其在列表中的位置。必须与`steps=true`一起使用。
`banner_instructions`<br /> （可选） | 返回与路径步骤关联的标题对象 (`true`) 或者不返回（`false`，默认值）。必须与`steps=true`一起使用。
`bearings`<br /> （可选） | 影响路径从路径点*开始*的方向。用于过滤路径点被放置路段的方向，确保车辆行驶路线的方向是正确的。执行请求，则提供第一个路径点的方位和半径值，其余值为空。必须与`radiuses`参数一起使用。每个路径点有2个逗号分隔的值：正北方向顺时针0到360度，以及角度偏离度数范围(建议值为45°或90°），格式为`{angle, degrees}`。如果赋值，方位列表的长度必须与坐标对的长度一致。但是可以跳过坐标，用`;`分隔符表示其位置。
`continue_straight`<br /> （可选） | 在离开中间路径点时，设置前进方向。如果设置为`true`，继续按同一方向前进。如果设置为`false`，则向反方向前进。`mapbox/driving`默认设置为`true`，for `mapbox/walking`和`mapbox/cycling`默认设置为`false`。
`exclude`<br/> （可选） | 路径中排除某种类型的道路。默认设置是不从所选配置文件中排除任何类型。每个配置文件中可用的`exclude`标志如下：<table><th>**配置文件**</th><th>**排除类型**</th><tr><td>`mapbox/driving`</td><td>`toll`，`motorway`或 `ferry`其一</td></tr><tr><td>`mapbox/driving-traffic`</td><td>`toll`，`motorway`或`ferry`其一</td></tr><tr><td>`mapbox/walking`</td><td>不支持排除</td></tr><tr><td>`mapbox/cycling`</td><td>`ferry`</td></tr></table>
`geometries`<br /> （可选） | 返回几何格式。属性值有： `geojson` (as [LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2)), [`polyline`](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) （默认值，精度为5的折线），[`polyline6`](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) （精度为6的折线）。
`language`<br /> （可选） | 返回多段导航命令文本的语言。请参考：[supported languages](#instructions-languages)。默认是`en`（英文）。 必须与`steps=true`一起使用。
`overview`<br /> （可选） | 返回几何的情况。选择`full`（最详细的几何），`simplified`（默认值，简化版精细几何），或者`false`（没有几何）。
`radiuses`<br /> （可选） | 坐标可以移动捕捉到路网上的最大距离，单位米。半径数量与请求坐标数目一致，每个用`;`分隔开。属性值可以为任意大于`0`的数或者字符串`unlimited`。如果在半径范围内找不到可规划的路，则返回`NoSegment`错误。
`roundabout_exits`<br /> （可选）| 在出环岛时发送指令（`true`）或不发送（`false`，默认值）。如果没有这个参数，环岛操作则为一个指令，同时包括进入环岛和退出环岛。当`roundabout_exits=true`时，该操作变成两个指令，一个用于进入环岛，一个用于退出环岛。
`steps`<br /> （可选） | 返回步骤和多段导航命令（`true`）或不返回（`false`，默认值）。
`voice_instructions`<br /> （可选） | 语音导航时，返回路径的[SSML](https://developer.amazon.com/docs/custom-skills/speech-synthesis-markup-language-ssml-reference.html)标记文本（`true`）或不返回（`false`，默认值）。 必须与`steps=true`一起使用。
`voice_units`<br /> （可选） | 指定要在语音指令的文本中返回的单位类型。选择`imperial`（默认值）或者`metric`。必须与`steps=true`和`voice_instructions=true`一起使用。
`waypoint_names`<br /> （可选）| 以分号分隔的自定义名称列表用作到达命令的标题和语音命令。属性值可以是任意字符，总数不超过500。如果赋值，`waypoint_names`列表的长度必须与坐标对的长度一致，但是可以跳过坐标，用`;`分隔符表示其位置。

查询字符串中无法识别的选项会导致`InvalidInput`错误。

**导航命令语言**<a id='instructions-languages'></a>

下表`language`参数是多段导航命令支持的语言代码。如果不支持该语言，则默认为`en`（英语）。

代码 | 语言
--- | ---
`da` | 丹麦语
`de` | 德语
`en` | 英语
`eo` | 世界语
`es` | 西班牙语
`es-ES` | 西班牙语（巴西）
`fi` | 芬兰语
`fr` | 法语
`he` | 希伯来语
`id` | 印度尼西亚语
`it` | 意大利语
`ko` | 韩语
`my` | 缅甸语
`nl` | 荷兰语
`no` | 挪威语（挪威）
`pl` | 波兰语
`pt-BR` | 葡萄牙语（巴西）
`pt-PT` | 葡萄牙（葡萄牙）
`ro` | 罗马尼亚
`ru` | 俄语
`sv` | 瑞典语
`tr` | 土耳其语
`uk` | 乌克兰
`vi` | 越南语
`zh-Hans` | 汉语 (简体)

#### 请求举例

```curl
# Request directions with no additional options
curl "https://api.mapbox.com/directions/v5/mapbox/cycling/-122.42,37.78;-77.03,38.91?access_token={your_access_token}"

# Request directions with steps and alternatives
curl "https://api.mapbox.com/directions/v5/mapbox/cycling/-122.42,37.78;-77.03,38.91?steps=true&alternatives=true&access_token={your_access_token}"

# Request directions with radiuses and a polyline6 response through multiple waypoints
curl "https://api.mapbox.com/directions/v5/mapbox/driving/13.43,52.51;13.42,52.5;13.41,52.5?radiuses=40;;100&geometries=polyline6&access_token={your_access_token}"

# Specify bearings and radiuses
curl "https://api.mapbox.com/directions/v5/mapbox/driving/13.43,52.51;13.42,52.5;13.43,52.5?bearings=60,45;;45,45&radiuses=100;100;100&access_token={your_access_token}"

# Request a route that approaches the destination on the curb of the driving side
curl "https://api.mapbox.com/directions/v5/mapbox/driving/13.43,52.51;13.43,52.5?approaches=unrestricted;curb&access_token={your_access_token}"

# Request directions with voice and banner instructions
curl "https://api.mapbox.com/directions/v5/mapbox/cycling/-122.42,37.78;-77.03,38.91?steps=true&voice_instructions=true&banner_instructions=true&voice_units=imperial&access_token={your_access_token}"

# Request directions with voice and banner instructions specifying waypoint_names
curl "https://api.mapbox.com/directions/v5/mapbox/cycling/-122.42,37.78;-77.03,38.91?steps=true&voice_instructions=true&banner_instructions=true&voice_units=imperial&waypoint_names=Home;Work&access_token={your_access_token}"
```

```python
service = Directions()
# The input waypoints are features, typically GeoJSON-like feature dictionaries
origin = {
    'type': 'Feature',
    'properties': {'name': 'Portland, OR'},
    'geometry': {
        'type': 'Point',
        'coordinates': [-122.7282, 45.5801]}}
destination = {
    'type': 'Feature',
    'properties': {'name': 'Bend, OR'},
    'geometry': {
        'type': 'Point',
        'coordinates': [-121.3153, 44.0582]}}
# The directions() method can be called with a list of features and the desired profile
response = service.directions([origin, destination], 'mapbox.driving')
```

```javascript
directionsClient
  .getDirections({
    waypoints: [
      {
        coordinates: [13.4301, 52.5109],
        approach: 'unrestricted'
      },
      {
        coordinates: [13.4265, 52.508]
      },
      {
        coordinates: [13.4194, 52.5072],
        bearing: [100, 60]
      }
    ]
  })
  .send()
  .then(response => {
    const directions = response.body;
  });
```

```bash
# This API is not available through the Mapbox CLI
```

```java
MapboxDirections client = MapboxDirections.builder()
  .accessToken("{your_access_token}")
  .origin(origin)
  .destination(destination)
  .overview(DirectionsCriteria.OVERVIEW_FULL)
  .profile(DirectionsCriteria.PROFILE_CYCLING)
  .build();
```

```objc
NSArray<MBWaypoint *> *waypoints = @[
    [[MBWaypoint alloc] initWithCoordinate:CLLocationCoordinate2DMake(38.9099711, -77.0361122) name:@"Mapbox"],
    [[MBWaypoint alloc] initWithCoordinate:CLLocationCoordinate2DMake(38.8977, -77.0365) name:@"White House"],
];
MBRouteOptions *options = [[MBRouteOptions alloc] initWithWaypoints:waypoints profileIdentifier:MBDirectionsProfileIdentifierCycling];
options.includesSteps = YES;

NSURLSessionDataTask *task = [directions calculateDirectionsWithOptions:options completionHandler:^(NSArray<MBWaypoint *> * _Nullable waypoints, NSArray<MBRoute *> * _Nullable routes, NSError * _Nullable error) {
    if (error) {
        NSLog(@"Error calculating directions: %@", error);
        return;
    }

    MBRoute *route = routes.firstObject;
    MBRouteLeg *leg = route.legs.firstObject;
    if (leg) {
        NSLog(@"Route via %@:", leg);

        NSLengthFormatter *distanceFormatter = [[NSLengthFormatter alloc] init];
        NSString *formattedDistance = [distanceFormatter stringFromMeters:leg.distance];

        NSDateComponentsFormatter *travelTimeFormatter = [[NSDateComponentsFormatter alloc] init];
        travelTimeFormatter.unitsStyle = NSDateComponentsFormatterUnitsStyleShort;
        NSString *formattedTravelTime = [travelTimeFormatter stringFromTimeInterval:route.expectedTravelTime];

        NSLog(@"Distance: %@; ETA: %@", formattedDistance, formattedTravelTime);

        for (MBRouteStep *step in leg.steps) {
            NSLog(@"%@", step.instructions);
            NSString *formattedDistance = [distanceFormatter stringFromMeters:step.distance];
            NSLog(@"— %@ —", formattedDistance);
        }
    }
}];
```

```swift
let waypoints = [
    Waypoint(
      coordinate: CLLocationCoordinate2D(latitude: 38.9099711, longitude: -77.0361122),
      name: "Mapbox"),
    Waypoint(
      coordinate: CLLocationCoordinate2D(latitude: 38.8977, longitude: -77.0365),
      name: "White House"),
]
let options = RouteOptions(waypoints: waypoints, profileIdentifier: .cycling)
options.includesSteps = true

let task = directions.calculate(options) { (waypoints, routes, error) in
    guard error == nil else {
        print("Error calculating directions: \(error!)")
        return
    }

    if let route = routes?.first, let leg = route.legs.first {
        print("Route via \(leg):")

        let distanceFormatter = LengthFormatter()
        let formattedDistance = distanceFormatter.string(fromMeters: route.distance)

        let travelTimeFormatter = DateComponentsFormatter()
        travelTimeFormatter.unitsStyle = .short
        let formattedTravelTime = travelTimeFormatter.string(from: route.expectedTravelTime)

        print("Distance: \(formattedDistance); ETA: \(formattedTravelTime!)")

        for step in leg.steps {
            print("\(step.instructions)")
            let formattedDistance = distanceFormatter.string(fromMeters: step.distance)
            print("— \(formattedDistance) —")
        }
    }
}
```

### 路径响应对象

Directions API的请求响应是一个JSON对象，它包括以下属性：

属性 | 说明
--- | ---
`code` | 指示响应状态的字符串。这是与HTTP状态代码不同的代码。正常有效响应的情况下，值将为`Ok`。否则，参照：[Directions API errors table](#directions-api-errors).
`waypoints` | [waypoint](#waypoint-object)对象的数组。每个点都是捕捉在道路和路网上输入的坐标对。点的顺序与坐标对输入顺序一致。
`routes` | 按推荐等级降序排列的[route](#route-object)对象数组。响应对象最多有2条路径。

#### 响应对象举例

```json
{
  "routes": [
    {
      "geometry": "mnn_Ick}pAfBiF`CzA",
      "legs": [
        {
          "summary": "Köpenicker Straße, Engeldamm",
          "weight": 44.4,
          "duration": 26.2,
          "steps": [
            {
              "intersections": [
                {
                  "out": 0,
                  "entry": [ true ],
                  "bearings": [ 125 ],
                  "location": [ 13.426579, 52.508068 ]
                },
                {
                  "out": 1,
                  "in": 2,
                  "entry": [ true, true, false ],
                  "bearings": [ 30, 120, 300 ],
                  "location": [ 13.426688, 52.508022 ]
                }
              ],
              "driving_side": "right",
              "geometry": "mnn_Ick}pAHUlAqDNa@",
              "mode": "driving",
              "maneuver": {
                "bearing_after": 125,
                "bearing_before": 0,
                "location": [ 13.426579, 52.508068 ],
                "modifier": "right",
                "type": "depart",
                "instruction": "Head southeast on Köpenicker Straße (L 1066)"
              },
              "ref": "L 1066",
              "weight": 35.9,
              "duration": 17.7,
              "name": "Köpenicker Straße (L 1066)",
              "distance": 98.1,
              "voiceInstructions": [
                {
                  "distanceAlongGeometry": 98.1,
                  "announcement": "Head southeast on Köpenicker Straße (L 1066), then turn right onto Engeldamm",
                  "ssmlAnnouncement": "<speak><amazon:effect name=\"drc\"><prosody rate=\"1.08\">Head southeast on Köpenicker Straße (L <say-as interpret-as=\"address\">1066</say-as>), then turn right onto Engeldamm</prosody></amazon:effect></speak>"
                },
                {
                  "distanceAlongGeometry": 83.1,
                  "announcement": "Turn right onto Engeldamm, then you will arrive at your destination",
                  "ssmlAnnouncement": "<speak><amazon:effect name=\"drc\"><prosody rate=\"1.08\">Turn right onto Engeldamm, then you will arrive at your destination</prosody></amazon:effect></speak>"
                }
              ],
              "bannerInstructions": [
                {
                  "distanceAlongGeometry": 98.1,
                  "primary": {
                    "text": "Engeldamm",
                    "components": [ { "text": "Engeldamm" } ],
                    "type": "turn",
                    "modifier": "right"
                  },
                  "secondary": null,
                  "then": null
                }
              ]
            },
            {
              "intersections": [
                {
                  "out": 2,
                  "in": 3,
                  "entry": [ false, true, true, false ],
                  "bearings": [ 30, 120, 210, 300 ],
                  "location": [ 13.427752, 52.50755 ]
                }
              ],
              "driving_side": "right",
              "geometry": "ekn_Imr}pARL\\T^RHDd@\\",
              "mode": "driving",
              "maneuver": {
                "bearing_after": 202,
                "bearing_before": 125,
                "location": [ 13.427752, 52.50755 ],
                "modifier": "right",
                "type": "turn",
                "instruction": "Turn right onto Engeldamm"
              },
              "weight": 8.5,
              "duration": 8.5,
              "name": "Engeldamm",
              "distance": 78.6,
              "voiceInstructions": [
                {
                  "distanceAlongGeometry": 27.7,
                  "announcement": "You have arrived at your destination",
                  "ssmlAnnouncement": "<speak><amazon:effect name=\"drc\"><prosody rate=\"1.08\">You have arrived at your destination</prosody></amazon:effect></speak>"
                }
              ],
              "bannerInstructions": [
                {
                  "distanceAlongGeometry": 78.6,
                  "primary": {
                    "text": "You will arrive at your destination",
                    "components": [ { "text": "You will arrive at your destination" } ],
                    "type": "arrive",
                    "modifier": "straight"
                  },
                  "secondary": {
                    "text": "Engeldamm",
                    "components": [ { "text": "Engeldamm" } ],
                    "type": "arrive",
                    "modifier": "straight"
                  }
                },
                {
                  "distanceAlongGeometry": 15,
                  "primary": {
                    "text": "You have arrived at your destination",
                    "components": [ { "text": "You have arrived at your destination" } ],
                    "type": "arrive",
                    "modifier": "straight"
                  },
                  "secondary": {
                    "text": "Engeldamm",
                    "components": [ { "text": "Engeldamm" } ]
                  }
                }
              ]
            },
            {
              "intersections": [
                {
                  "in": 0,
                  "entry": [ true ],
                  "bearings": [ 25 ],
                  "location": [ 13.427292, 52.506902 ]
                }
              ],
              "driving_side": "right",
              "geometry": "cgn_Iqo}pA",
              "mode": "driving",
              "maneuver": {
                "bearing_after": 0,
                "bearing_before": 205,
                "location": [ 13.427292, 52.506902 ],
                "type": "arrive",
                "instruction": "You have arrived at your destination"
              },
              "weight": 0,
              "duration": 0,
              "name": "Engeldamm",
              "distance": 0,
              "voiceInstructions": [ ],
              "bannerInstructions": [ ]
            }
          ],
          "distance": 176.7
        }
      ],
      "weight_name": "routability",
      "weight": 44.4,
      "duration": 26.2,
      "distance": 176.7
    }
  ],
  "waypoints": [
    {
      "name": "Köpenicker Straße",
      "location": [ 13.426579, 52.508068 ]
    },
    {
      "name": "Engeldamm",
      "location": [ 13.427292, 52.506902 ]
    }
  ],
  "code": "Ok",
  "uuid": "cjd51uqn5005447p8nte2zc4w"
}
```

### 路点对象

Directions API查询的响应主体包含一个**路点对象**， 输入的坐标将会被捕捉到路网上，一个路点对象包含以下属性：

属性 | 描述
--- | ---
`name` | 一个字符串，其中包含输入坐标捕捉到的道路或路径的名称。
`location` |一个数组包括输入坐标捕捉到的经纬度 `[longitude, latitude]`。

#### 路点对象举例

```json
{
    "name": "Kirkjubøarvegur",
    "location": [ -6.80897, 62.000075 ]
}
```

### 路线对象

Directions API查询的响应主体包含一个数组组成的**路线对象**。一个路线对象描述了一条通过多个路点的路线。路线对象包括以下属性:

属性 | 描述
--- | ---
`duration` | 一个浮点数，表示通过路点的估计行程时间（以秒为单位）。
`distance` | 一个浮点数，表示通过航点的行程距离（以米为单位）。
`weight_name` | 一个字符串表示选择的权重。 默认值是 `routability`，这是基于持续时间的，对不太理想的路线有额外的惩罚。
`weight` |  一个浮点数，表述以`weight_name`的单位表示的权重。
`geometry` | 基于查询参数： `geometries` ，这是一个 [GeoJSON LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2) 或 [Polyline string](https://developers.google.com/maps/documentation/utilities/polylinealgorithm). 基于查询参数 `overview` ，这可能是一个完整的路线几何体 (`full`), 一个在能显示完整路径的缩放比例上的简化几何图形 (`simplified`), 或者没有 (`false`).
`legs` | 一个数组关于： [route leg](#routeleg-object) 。
`voiceLocale` | 一个字符串，设定用于语音指令的语言环境. 默认是 `en` (英语). 可以改为任何 [accepted instruction language](#instructions-languages). `voiceLocale` 仅在 `voice_instructions=true`时出现。

#### 路线对象举例

```json
{
    "duration": 88.4,
    "distance": 830.4,
    "weight": 88.4,
    "weight_name": "routability",
    "geometry": "oklyJ`{ph@yBuY_F{^_FxJoBrBs@d@mAT",
    "legs": [ ],
    "voiceLocale": "en"
}
```

### 路段对象

路程对象中嵌套**路段**对象，用于描述路程中的每一段路，数量比输入的坐标少一个。路段对象包括以下属性:

属性 | 描述
--- | ---
`distance` | 一个数字，表示通过的路点之间的距离（以米为单位）。
`duration` | 一个数字，表示通过的路点预期要花费的时间（以秒为单位）。
`steps` | 基于可选的 `steps` 参数，可能是一个[route step](#routestep-object) 的数组 (当`steps=true`) 或为空（当`steps=false`，默认）。
`summary` | 一个总结路线的字符串。
`annotation` | 注释对象，其中包含有关路线几何图形的每个线段的其他详细信息。 注释字段中的每个条目对应于路线几何的坐标。 有关`annotation`对象的更多信息，请参见下表：

`annotation` | 描述
--- | ---
`distance` | 每对坐标之间的距离，以米为单位。
`duration` | 每对坐标之间的时间花销，以秒为单位。
`speed` | 每对坐标之间的速度，以米/秒为单位。
`congestion` | 拥堵程度, 用 `severe`，`heavy`，`moderate`，`low`或`unknown`描述。每个条目对应于一个路段. 对于除“mapbox / driving-traffic”以外的任何配置文件，将返回`unknown`的列表。 如果路线很长，也会返回`unknown`的列表。

#### 路段对象举例

```json
{
    "annotation": {
        "distance": [
            4.294596842089401,
            5.051172053200946,
            5.533254065167979,
            6.576513793849532,
            7.4449640160938015,
            8.468757534990829,
            15.202780313562714,
            7.056346577326572
        ],
        "duration": [
            1,
            1.2,
            2,
            1.6,
            1.8,
            2,
            3.6,
            1.7
        ],
        "speed": [
            4.3,
            4.2,
            2.8,
            4.1,
            4.1,
            4.2,
            4.2,
            4.2
        ],
        "congestion": [
            "low",
            "moderate",
            "moderate",
            "moderate",
            "heavy",
            "heavy",
            "heavy",
            "heavy"
        ]
    },
    "duration": 14.3,
    "weight": 14.3,
    "distance": 53.4,
    "steps": [],
    "summary": ""
}
```

### 行路步骤对象

在路段对象中，嵌套的**行路步骤**对象包括一个[step maneuver](#stepmaneuver-object) 对象以及如何前往下一段路的信息：

属性 | 描述
--- | ---
`maneuver` | 一个[step maneuver](#stepmaneuver-object) 对象。
`distance` | 一个数字，表示从现在行为到下一个行路步骤的行进距离（以米为单位）。
`duration` | 一个数字，表示从现在行为到下一个行路步骤的预期时间（以秒为单位）。
`geometry` | 基于 `geometries` 参数，会是 [GeoJSON LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2) 或 [Polyline string](https://developers.google.com/maps/documentation/utilities/polylinealgorithm)，表示从此行路步骤到下一个步骤的完整路线几何信息。
`name` | 一个字符串，其中包含构成行路步骤一部分的道路或路径的名称。
`ref` | 任何与 [road designations](https://en.wikipedia.org/wiki/Road_designation_or_abbreviation) 关联的道路或线路，引导这一步到下一步的行为。 如果关联到 [multiple road designations](https://en.wikipedia.org/wiki/Concurrency_%28road%29) ，则他们以分号分隔。通常由字母路网代码（标识道路类型或编号系统），空格或连字符以及 [route number](https://en.wikipedia.org/wiki/Route_number)组成。 可选属性，当数据可用时才存在。<br>**Note:** 路网代码不一定是全局唯一的，不应该被如此对待。路线编号可能不唯一地标识给定网络内的道路。
`destinations` | 一个字符串，其中包含旅行所经过的道路或路径的目的地。 
`exits` | 一个字符串，表示离开此道路时的出口编号或名称。可选属性，当数据可用时才存在。
`driving_side` | 此位置的合法驾驶方向， `left` 或 `right`。
`mode` | 一个字符串，表示交通方式。 <table><tr><th>大纲</th><th>可能的值</th></tr><tr><td>`mapbox/driving` </td><td>`driving`, `ferry`, `unaccessible`</td></tr><tr><td>`mapbox/walking`</td><td>`walking`, `ferry`, `unaccessible`</td></tr><tr><td>`mapbox/cycling`</td><td>`cycling`, `walking`, `ferry`, `train`, `unaccessible`</td></tr></table>
`pronunciation` | 一个字符串，包含[IPA](https://en.wikipedia.org/wiki/International_Phonetic_Alphabet) 语音转录的字符串，表示如何在`name`属性中发音。如果没有发音数据，则省略。
`intersections` | 表示沿着行路步骤的所有交叉点的对象数组。 有关`intersectionctions`数组的更多信息，请参见下表：


`intersections` | 描述
--- | ---
`location` | 一个 `[longitude, latitude]` 描述转弯点的位置。
`bearings` | 一个列表，列出交叉路口可用的方向值，描述交叉路口的所有可用道路。
`classes` | 一组字符串，表示离开交叉路口的道路类别。  <table><tr><th>可能值</th><th>描述</th></tr><tr><td>`toll`</td><td>连接到收费公路</td></tr><tr><td>`ferry`</td><td>连接到轮渡</td></tr><tr><td>`restricted`</td><td>有通行限制</td></tr><tr><td>`motorway`</td><td>连接到公路</td></tr><tr><td>`tunnel`</td><td>连接到隧道</td></tr></table>
`entry` | 一个关于入口标志的列表，1:1 对应 `bearings`. 如果为`true`，则表示可以在有效路线上进入相应的道路，如果为`false`，转向相应的道路将违反限制。
`in` | `bearing`和`entry`数组中的索引。用于计算转弯之前的轴向。即，在操纵/通过交叉点之前从真北到行进方向的顺时针角度。为了使轴朝向行驶方向，轴向必须旋转180°。该值不用于离场行为。
`out` | `bearing`和`entry`数组中的索引。用于提取转弯之后的轴向。即，在操纵/通过交叉点之后从真北到行进方向的顺时针角度。到达行为不提供该值。
`lanes` | 一个[lane](#lane-object) 数组，代表交叉路口的可用转弯车道。如果没有可用于交叉路口的车道信息，则不会出现`lanes`属性。

#### 行路步骤对象举例

```json
{
    "intersections": [
      {
        "out": 1,
        "location": [ 13.424671, 52.508812 ],
        "bearings": [ 120, 210, 300 ],
        "entry": [ false, true, true ],
        "in": 0,
        "lanes": [
          {
            "valid": true,
            "indications": [ "left" ]
          },
          {
            "valid": true,
            "indications": [ "straight" ]
          },
          {
            "valid": false,
            "indications": [ "right" ]
          }
        ]
      }
    ],
    "geometry": "asn_Ie_}pAdKxG",
    "maneuver": {
      "bearing_after": 202,
      "type": "turn",
      "modifier": "left",
      "bearing_before": 299,
      "location": [ 13.424671, 52.508812 ],
      "instruction": "Turn left onto Adalbertstraße"
    },
    "duration": 59.1,
    "distance": 236.9,
    "driving_side": "right",
    "weight": 59.1,
    "name": "Adalbertstraße",
    "mode": "driving"
}
```

### 步骤操作对象

路径步骤对象包含嵌套的**步骤操作**对象，其中包含以下属性：

属性 | 描述
------------------| ------------------
`bearing_before`  | `0`到`360`之间的数字，表示在操作 _之前_ 这一瞬间从真北到行进方向的顺时针角度。
`bearing_after`   | `0`到`360`之间的数字，表示在操作 _之后_ 这一瞬间从真北到行进方向的顺时针角度。
`instruction`     | 关于如何执行返回操作的人类可读指令。
`location`        | 一个 `[longitude, latitude]` 数组，对应操作点的位置。
`modifier`        | 一个可选的字符串，表示操纵的方向变化。每个 `modifier` 的含义取决于 `type` 参数。<table><tr><th>可能值</th><th>描述</th></tr><tr><td>`uturn`</td><td>表示掉头，在同一条路上时 `type` 可能是 `turn` 或 `continue` 。</td></tr><tr><td>`sharp right`</td><td>右急转弯</td></tr><tr><td>`right`</td><td>正常右转弯。</td></tr><tr><td>`slight right`</td><td>稍向右转。</td></tr><tr><td>`straight`</td><td>方向不变。</td></tr><tr><td>`slight left`</td><td>稍向左转。</td></tr><tr><td>`left`</td><td>正常左转弯。</td></tr><tr><td>`sharp left`</td><td>左急转弯。</td></tr></table>
`type`            | 表示操作类型的字符串。 查看 [maneuver types table](#maneuver-types)中的操作类型的完整列表。

- 如果不提供 `modifier` ，`type` 会被限制在 `depart` 或 `arrive`。
- 如果源位置或目标位置足够接近 `depart` 或 `arrive` 位置， `modifier` 将不会提供。

**操作类型**<a id='maneuver-types'></a>

`type` | 描述
 --- | ---
`turn` | 基于`modifier`的基本转向。
`new name` | 发生变化后的道路名称（在强制转弯后）。
`depart` | 表示离开一个路段。 `modifier` 的值表示出发点相对于当前行进方向的位置。
`arrive` | 表示到达一个路段的目的地。 `modifier` 表示到达点相对于当前行进方向的位置。
`merge` | 融合到一条街上。
`on ramp` | 经由坡道进入高速公路。
`off ramp` | 经由坡道离开高速公路。
`fork` | 从岔道的左侧或右侧。
`end of road` | 道路在T字路口结束。
`continue` | 转弯后继续在街上行驶。
`roundabout` | 通过环岛，在[route step](#routestep-object)中有个附加属性`exit`，包含驶离的编号。 `modifier` 指定进入环岛的方向。
`rotary` | 一个环形路。 虽然与较大版本的环形交叉口非常相似，但它并不一定遵循环形交通规则。可以提供 `rotary_name` 、 `rotary_pronunciation` 两个属性之一或两者皆有。通过 [route step](#routestep-object) 定位并有个附加属性`exit`。
`roundabout turn` |一个被视为十字路口的小型环岛。
`notification` | 表示驾驶情况的变化，例如`mode`从`driving`变为`ferry`。
`exit roundabout` | 表示驶离环岛的操作。 除非在请求中提供`roundabout_exits = true`查询参数，否则不会出现在结果中。
`exit rotary` | 表示驶离小环路的操作。 除非在请求中提供`roundabout_exits = true`查询参数，否则不会出现在结果中。

**Note:** 将来可能会引入新属性（可能取决于`type`），即使没有更新API版本。

#### 步骤操作对象举例

```json
{
    "bearing_before": 299,
    "bearing_after": 202,
    "type": "turn",
    "modifier": "left",
    "location": [ 13.424671, 52.508812 ],
    "instruction": "Turn left onto Adalbertstraße"
}
```

### 车道对象

行路步骤对象嵌套了**车道对象**。车道对象描述了交叉路口的可用转弯车道。 车道从左到右依次在街道上提供。

 属性 | 描述
--- | ---
 `valid` | 表示是否可以在此车道完成操作(`true`) 或 (`false`). 例如，如果一边有四个车道对象且前两个有效，则驾驶员可以选择左侧两个车道中的任何一个并留在路线上。
 `indications` | 一系列转弯车道的标识，可能有多个。例如，转弯车道可以有一个箭头指向左侧，另一个箭头指向右侧。

#### 车道对象举例

```json
{
    "valid": true,
    "indications": [
        "left"
    ]
}
```

### 语音指令对象

如果存在可选的`voice_instructions = true`查询参数，则行路步骤对象包含嵌套的**语音指令对象**。语音指令对象包含应该显示的文本，以及应该发出操作指令的距离。语言指令是行路步骤的子项，应该在此步骤之中说出他们，但实际上他们指的是 _下一步_ 该做的操作。

属性 | 描述
--- | ---
`distanceAlongGeometry` | 一个浮点数，表示语音指令到即将到来的操作的距离（以米为单位）。
`announcement` | 包含语音指令文本的字符串。
`ssmlAnnouncement` | 带有SSML标记的字符串，用于正确的文本和发音。 被设计和[Amazon Polly](https://aws.amazon.com/polly/)一起使用. SSML标记可能无法与其他文本语音转换引擎一起使用。

#### 语音指令对象举例

```json
{
    "distanceAlongGeometry": 375.7,
    "announcement": "In a quarter mile, take the ramp on the right towards I 495: Baltimore",
    "ssmlAnnouncement": "<speak><amazon:effect name=\"drc\"><prosody rate=\"1.08\">In a quarter mile, take the ramp on the right towards <say-as interpret-as=\"address\">I-495</say-as>: Baltimore</prosody></amazon:effect></speak>"
}
```

### 横幅指令对象

如果存在可选的`banner_instructions = true`查询参数，则路径步骤对象包含嵌套的**横幅指令对象**。 横幅指令对象包含横幅的内容，该横幅应显示为路径的附加视觉指导。 指令横幅是行路步骤的子项，应该在此步骤之中展现他们，但实际上他们指的是 _下一步_ 该做的操作。

属性 | 说明
--- | ---
`distanceAlongGeometry` | 浮点型，表示距离下一个操作多远时指令横幅开始显示，单位米。一次只能显示一个横幅。
`primary` | 向用户展示最重要的内容。文字位于顶部，字号更大。
`secondary` | 对可视化引导有用的其他信息。此文本略小于`primary`文本。 可以为`null`。
`sub` | 驾驶者需要被告知的某些内容。如果步骤很短，可以包括_下一条_操作的信息（即将操作的下一个）。如果车道信息可用，则优先于 _下一条_ 操作的信息。

不同类型（`primary`，`secondary`，和`sub`）标题包括以下属性：

属性 | 说明
--- | ---
`text` | 包含应显示的所有文本的字符串。
`type` | 操作类型。可以与`modifier` (和`degrees`，如果有环岛）一起使用，用于图标显示。可能的值：`turn`，`merge`，`depart`，`arrive`，`fork`，`off ramp`和`roundabout`。
`modifier`<br>（可选） | The modifier for the maneuver. 可以与`type`（和`degrees`，如果有环岛）一起使用，用于图标显示。 <table><tr><th>可能的值</th><th>说明</th></tr><tr><td>`uturn`</td><td>掉头</td></tr><tr><td>`sharp right`</td><td>右急转弯</td></tr><tr><td>`right`</td><td>正常右转弯</td></tr><tr><td>`slight right`</td><td>稍向右转</td></tr><tr><td>`straight`</td><td>方向不变</td></tr><tr><td>`slight left`</td><td>稍向左转</td></tr><tr><td>`left`</td><td>正常左转弯</td></tr><tr><td>`sharp left`</td><td>左急转弯</td></tr></table>
`degrees`<br>（可选）| 离开环岛时的度数，假设`180`表示直走通过环岛。
`driving_side`<br>（可选） | 字符串，表示行驶在道路的哪一侧。可以是`left`或者`right`。
`components` | 与对象一起组成标题。包括辅助可视化排版其他信息。它包括以下属性：

`components` | 说明
--- | ---
`type` | 包含有关组件的更多上下文的字符串，可帮助进行可视标记和显示选择。 如果组件的类型未知，则应将其视为文本。 <table><tr><th>可能值</th><th>描述</th></tr><tr><td>`text`</td><td>默认。 表示此文本是说明的一部分，没有其他类型。</td></tr><tr><td>`icon` </td><td>这是可以用imageBaseURL图标替换的文本。</td></tr><tr><td>`delimiter`</td><td>这是可以删除的文本，如果要渲染图标，则应删除该文本。</td></tr><tr><td>`exit-number`</td><td>表示操作时的出口编号。</td></tr><tr><td>`exit`</td><td>以本地语言提供 _exit_ 的单词。</td></tr><tr><td>`lane`</td><td>指示哪些车道可用于完成操作。</td></tr></table> 后续新类型的引入不被认为是一个突破性的变化。
`text` | 父对象的`text`的子字符串，可能具有与之关联的附加上下文。
`abbr`<br>(可选) | `text`的缩写形式。 如果存在，则还会有一个`abbr_priority`值。 有关使用`abbr`和`abbr_priority`的示例，请参阅[abbreviation examples](#abbreviation-examples) 表。
`abbr_priority`<br>(可选) | 一个整数，表示应使用缩写`abbr`代替`text`的顺序。最高优先级时`0`，更高的整形值意味着更低的优先级。整数值之间没有间隙。多个组件可以具有相同的`abbr_priority`。当发生这种情况时，所有具有相同`abbr_priority`的`components`应该同时缩写。找不到更大的`abbr_priority`值意味着该字符串是完全缩写的。
`imageBaseURL`<br>(可选) | 一个字符串，指向用于替换文本的图标。
`directions` | 一个数组，指示您可以从一个车道走哪些方向（左，右或直行）。如果值时['left', 'straight']，司机可以从那条车道直行或向左。当`components.type`是`lane`时才显示。
`active` |一个布尔值，告诉您该车道是否可用于完成即将到来的操作。 如果多个车道处于激活状态，那么它们都可以用于完成即将到来的操作。当`components.type`是`lane`时才显示。

**缩写举例**<a id='abbreviation-examples'></a>

`text`             | `abbr`       | `abbr_priority`
-------------------|--------------|--------------------
North              | N            | 1
Franklin Drive     | Franklin Dr  | 0

鉴于上表中的`components`，可能的缩写按顺序排列：
- North Franklin Dr
- N Franklin Dr

#### 横幅指令对象案例

```json
{
    "distanceAlongGeometry": 100,
    "primary": {
        "type": "turn",
        "modifier": "left",
        "text": "I 495 North / I 95",
        "components": [
          {
            "text": "I 495",
            "imageBaseURL": "https://s3.amazonaws.com/mapbox/shields/v3/i-495",
            "type": "icon"
          },
          {
            "text": "North",
            "type": "text",
            "abbr": "N",
            "abbr_priority": 0
          },
          {
            "text": "/",
            "type": "delimiter"
          },
          {
            "text": "I 95",
            "imageBaseURL": "https://s3.amazonaws.com/mapbox/shields/v3/i-95",
            "type": "icon"
          }
        ]
    },
    "secondary": {
        "type": "turn",
        "modifier": "left",
        "text": "Baltimore / Northern Virginia",
        "components": [
          {
            "text": "Baltimore",
            "type": "text"
          },
          {
            "text": "/",
            "type": "text"
          },
          {
            "text": "Northern Virginia",
            "type": "text"
          }
      ]
    },
    "sub": {
        "text": "",
        "components": [
          {
            "text": "",
            "type": "lane",
            "directions": ["left"],
            "active": true
          },
          {
            "text": "",
            "type": "lane",
            "directions": ["left", "straight"],
            "active": true
          },
          {
            "text": "",
            "type": "lane",
            "directions": ["right"],
            "active": false
          }
        ]
    }
}
```

### 路径规划API错误

出错时，服务器会使用不同的HTTP状态代码进行响应。对于低于`500`的HTTP状态代码，JSON响应主体包含`code`属性，客户端程序可以使用它来管理控制流。响应主体还可以包括具有人类可读的错误解释的`message`属性。

如果发生服务器错误，HTTP状态代码将为`500`或更高，并且响应将不包含`code`属性。

响应主体`code` | HTTP状态代码 | 描述
--- | --- |---
`Ok` | `200` | 正常成功情况。
`NoRoute` | `200` | 根据给定的坐标找不到路径。检查不可能的路线（例如，没有渡轮连接的海洋上的路线）。
`NoSegment` | `200` | 给定坐标无法匹配到路段。 检查距离道路太远的坐标。
`ProfileNotFound` | `404` | 使用有效的配置文件，如 [list of routing profiles](#directions)中所述。
`InvalidInput` | `422` | 给定的请求无效。响应中的`message`将保留无效输入的解释。
