## Directions
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

**Restrictions and limits**
**限制**

- 使用配置文件`driving`，`walking`和 `cycling`的请求可沿路径指定最多25个点（输入坐标将捕捉路网）。
- 使用配置文件`driving-traffic`的请求可沿路径指定最多3个点。
- [supported geographies](./pages/traffic-countries.html) 提供了`driving-traffic`配置文件的交通信息。该配置文件在没有交通信息的地区则返回 `driving`文件。
- 每分钟最多支持60次请求。
- 所有点之间距离最多10000公里。

### Retrieve directions
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
`approaches`<br /> （可选） | A semicolon-separated list indicating the side of the road from which to approach waypoints in a requested route. Accepts `unrestricted`（默认值, route can arrive at the waypoint from either side of the road）或者 `curb` (route will arrive at the waypoint on the `driving_side` of the region). 如果提供，方法的数量必须与路径点数量相同。但是，你可以跳过坐标并通过`;`分隔符表示其在列表中的位置。必须与`steps=true`一起使用。
`banner_instructions`<br /> （可选） | Whether to return banner objects associated with the route steps (`true`) or not (`false`, default). Must be used in conjunction with `steps=true`.
`bearings`<br /> （可选） | Influences the direction in which a route *starts* from a waypoint. Used to filter the road segment the waypoint will be placed on by direction. This is useful for making sure the new routes of rerouted vehicles continue traveling in their current direction. A request that does this would provide bearing and radius values for the first waypoint and leave the remaining values empty. Must be used in conjunction with the `radiuses` parameter. Takes 2 comma-separated values per waypoint: an angle clockwise from true north between 0 and 360, and the range of degrees by which the angle can deviate (recommended value is 45° or 90°), formatted as `{angle, degrees}`. If provided, the list of bearings must be the same length as the list of coordinates. However, you can skip a coordinate and show its position in the list with the `;` separator.
`continue_straight`<br /> （可选） | Sets the allowed direction of travel when departing intermediate waypoints. If `true`, the route will continue in the same direction of travel. If `false`, the route may continue in the opposite direction of travel. Defaults to `true` for `mapbox/driving` and `false` for `mapbox/walking` and `mapbox/cycling`.
`exclude`<br/> （可选） | Exclude certain road types from routing. The default is to not exclude anything from the profile selected. The following `exclude` flags are available for each profile:<table><th>**Profile**</th><th>**Available excludes**</th><tr><td>`mapbox/driving`</td><td>One of `toll`, `motorway`, or `ferry`</td></tr><tr><td>`mapbox/driving-traffic`</td><td>One of `toll`, `motorway`, or `ferry`</td></tr><tr><td>`mapbox/walking`</td><td>No excludes supported</td></tr><tr><td>`mapbox/cycling`</td><td>`ferry`</td></tr></table>
`geometries`<br /> （可选） | 返回几何格式。属性值有： `geojson` (as [LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2)), [`polyline`](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) （默认值，精度为5的折线），[`polyline6`](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) （精度为6的折线）。
`language`<br /> （可选） | The language of returned turn-by-turn text instructions. See [supported languages](#instructions-languages). The default is `en` (English). Must be used in combination with `steps=true`.
`overview`<br /> （可选） | The type of returned overview geometry. Can be `full` (the most detailed geometry available), `simplified` (default, a simplified version of the full geometry), or `false` (no overview geometry).
`radiuses`<br /> （可选） | The maximum distance a coordinate can be moved to snap to the road network in meters. There must be as many radiuses as there are coordinates in the request, each separated by `;`. Values can be any number greater than `0` or the string `unlimited`. A `NoSegment` error is returned if no routable road is found within the radius.
`roundabout_exits`<br /> （可选）| Whether to emit instructions at roundabout exits (`true`) or not (`false`, default). Without this parameter, roundabout maneuvers are given as a single instruction that includes both entering and exiting the roundabout. With `roundabout_exits=true`, this maneuver becomes two instructions, one for entering the roundabout and one for exiting it.
`steps`<br /> （可选） | Whether to return steps and turn-by-turn instructions (`true`) or not (`false`, default).
`voice_instructions`<br /> （可选） | Whether to return [SSML](https://developer.amazon.com/docs/custom-skills/speech-synthesis-markup-language-ssml-reference.html) marked-up text for voice guidance along the route (`true`) or not (`false`, default). Must be used in conjunction with `steps=true`.
`voice_units`<br /> （可选） | Specify which type of units to return in the text for voice instructions. Can be `imperial` (default) or `metric`. Must be used in conjunction with be used in conjunction with `steps=true` and `voice_instructions=true`.
`waypoint_names`<br /> （可选）| A semicolon-separated list of custom names for coordinates used for the arrival instruction in banners and voice instructions. Values can be any string, and the total number of all characters cannot exceed 500. If provided, the list of `waypoint_names` must be the same length as the list of coordinates, but you can skip a coordinate and show its position with the `;` separator.

Unrecognized options in the query string result in an `InvalidInput` error.

**Instructions language**<a id='instructions-languages'></a>
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

#### Example request
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

### Directions response object
### 路径响应对象

Directions API的请求响应是一个JSON对象，它包括以下属性：

属性 | 说明
--- | ---
`code` | 指示响应状态的字符串。这是与HTTP状态代码不同的代码。正常有效响应的情况下，值将为`Ok`。否则，参照：[Directions API errors table](#directions-api-errors).
`waypoints` | [waypoint](#waypoint-object)对象的数组。每个点都是捕捉在道路和路网上输入的坐标对。点的顺序与坐标对输入顺序一致。
`routes` | 按推荐等级降序排列的[route](#route-object)对象数组。响应对象最多有2条路径。

#### Example response object
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

### Waypoint object
### 路点对象

The response body of a Directions API query contains a **waypoint object**, the input coordinates snapped to the roads network. A waypoint object contains the following properties:
Directions API查询的响应主体包含一个**路点对象**， 输入的坐标将会被捕捉到路网上，一个路点对象包含以下属性：

Property | Description
--- | ---
`name` | A string with the name of the road or path to which the input coordinate has been snapped.
`location` | An array containing the `[longitude, latitude]` of the snapped coordinate.

属性 | 描述
--- | ---
`name` | 一个字符串，其中包含输入坐标捕捉到的道路或路径的名称。
`location` |一个数组包括输入坐标捕捉到的经纬度 `[longitude, latitude]`。

#### Example waypoint object
#### 路点对象举例

```json
{
    "name": "Kirkjubøarvegur",
    "location": [ -6.80897, 62.000075 ]
}
```

### Route object
### 路线对象

The response body of a Directions API query also contains an array of **route objects**. A route object describes a route through multiple waypoints. A route object contains the following properties:
Directions API查询的响应主体包含一个数组组成的**路线对象**。一个路线对象描述了一条通过多个路点的路线。路线对象包括以下属性:

Property | Description
--- | ---
`duration` | A float indicating the estimated travel time through the waypoints in seconds.
`distance` | A float indicating the distance traveled through the waypoints in meters.
`weight_name` | A string indicating which weight was used. The default is `routability`, which is duration-based, with additional penalties for less desirable maneuvers.
`weight` | A float indicating the weight in units described by `weight_name`.
`geometry` | Depending on the `geometries` query parameter, this is either a [GeoJSON LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2) or a [Polyline string](https://developers.google.com/maps/documentation/utilities/polylinealgorithm). Depending on the `overview` query parameter, this is the complete route geometry (`full`), a simplified geometry to the zoom level at which the route can be displayed in full (`simplified`), or is not included (`false`).
`legs` | An array of [route leg](#routeleg-object) objects.
`voiceLocale` | A string of the locale used for voice instructions. Defaults to `en` (English). Can be any [accepted instruction language](#instructions-languages). `voiceLocale` is only present in the response when `voice_instructions=true`.

属性 | 描述
--- | ---
`duration` | 一个浮点数，表示通过路点的估计行程时间（以秒为单位）。
`distance` | 一个浮点数，表示通过航点的行程距离（以米为单位）。
`weight_name` | 一个字符串表示选择的权重。 默认值是 `routability`，这是基于持续时间的，对不太理想的路线有额外的惩罚。
`weight` |  一个浮点数，表述以`weight_name`的单位表示的权重。
`geometry` | 基于查询参数： `geometries` ，这是一个 [GeoJSON LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2) 或 [Polyline string](https://developers.google.com/maps/documentation/utilities/polylinealgorithm). 基于查询参数 `overview` ，这可能是一个完整的路线几何体 (`full`), 一个在能显示完整路径的缩放比例上的简化几何图形 (`simplified`), 或者没有 (`false`).
`legs` | 一个数组关于： [route leg](#routeleg-object) 。
`voiceLocale` | 一个字符串，设定用于语音指令的语言环境. 默认是 `en` (英语). 可以改为任何 [accepted instruction language](#instructions-languages). `voiceLocale` 仅在 `voice_instructions=true`时出现。

#### Example route object
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

### Route leg object
### 路段对象

A route object contains a nested **route leg** object for each leg of the journey, which is one fewer than the number of input coordinates. A route leg object contains the following properties:
路程对象中嵌套**路段**对象，用于描述路程中的每一段路，数量比输入的坐标少一个。路段对象包括以下属性:

Property | Description
--- | ---
`distance` | A number indicating the distance traveled between waypoints in meters.
`duration` | A number indicating the estimated travel time between waypoints in seconds.
`steps` | Depending on the optional `steps` parameter, either an array of [route step](#routestep-object) objects (`steps=true`) or an empty array (`steps=false`, default).
`summary` | A string summarizing the route.
`annotation` | An annotations object that contains additional details about each line segment along the route geometry. Each entry in an annotations field corresponds to a coordinate along the route geometry. See the following table for more information about the `annotation` object:

属性 | 描述
--- | ---
`distance` | 一个数字，表示通过的路点之间的距离（以米为单位）。
`duration` | 一个数字，表示通过的路点预期要花费的时间（以秒为单位）。
`steps` | 基于可选的 `steps` 参数，可能是一个[route step](#routestep-object) 的数组 (当`steps=true`) 或为空（当`steps=false`，默认）。
`summary` | 一个总结路线的字符串。
`annotation` | 注释对象，其中包含有关路线几何图形的每个线段的其他详细信息。 注释字段中的每个条目对应于路线几何的坐标。 有关`annotation`对象的更多信息，请参见下表：

`annotation` | Description
--- | ---
`distance` | The distance between each pair of coordinates in meters.
`duration` | The duration between each pair of coordinates in seconds.
`speed` | The speed between each pair of coordinates in meters per second.
`congestion` | The level of congestion, described as `severe`, `heavy`, `moderate`, `low` or `unknown`, between each entry in the array of coordinate pairs in the route leg. For any profile other than `mapbox/driving-traffic` a list of `unknown`s will be returned. A list of `unknown`s will also be returned if the route is very long.

`annotation` | 描述
--- | ---
`distance` | 每对坐标之间的距离，以米为单位。
`duration` | 每对坐标之间的时间花销，以秒为单位。
`speed` | 每对坐标之间的速度，以米/秒为单位。
`congestion` | 拥堵程度, 用 `severe`，`heavy`，`moderate`，`low`或`unknown`描述。每个条目对应于一个路段. 对于除“mapbox / driving-traffic”以外的任何配置文件，将返回`unknown`的列表。 如果路线很长，也会返回`unknown`的列表。



#### Example route leg object
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

### Route step object
### 行路步骤对象

In a route leg object, a nested **route step** object includes one [step maneuver](#stepmaneuver-object) object as well as information about travel to the following route step:
在路段对象中，嵌套的**行路步骤**对象包括一个[step maneuver](#stepmaneuver-object) 对象以及如何前往下一段路的信息：

Property | Description
--- | ---
`maneuver` | One [step maneuver](#stepmaneuver-object) object.
`distance` | A number indicating the distance traveled in meters from the maneuver to the next route step.
`duration` | A number indicating the estimated time traveled in seconds from the maneuver to the next route step.
`geometry` | Depending on the `geometries` parameter, this is a [GeoJSON LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2) or a [Polyline string](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) representing the full route geometry from this route step to the next route step.
`name` | A string with the name of the road or path that forms part of the route step.
`ref` | Any [road designations](https://en.wikipedia.org/wiki/Road_designation_or_abbreviation) associated with the road or path leading from this step’s maneuver to the next step’s maneuver. If [multiple road designations](https://en.wikipedia.org/wiki/Concurrency_%28road%29) are associated with the road, they are separated by semicolons. Typically consists of an alphabetic network code (identifying the road type or numbering system), a space or hyphen, and a [route number](https://en.wikipedia.org/wiki/Route_number). Optionally included, if data is available. <br>**Note:** A network code is not necessarily globally unique, and should not be treated as though it is. A route number may not uniquely identify a road within a given network.
`destinations` | A string with the destinations of the road or path along which the travel proceeds. Optionally included, if data is available.
`exits` | A string with the exit numbers or names of the road or path. Optionally included, if data is available.
`driving_side` | The legal driving side at the location for this step. Either `left` or `right`.
`mode` | A string indicating the mode of transportation. <table><tr><th>Profile</th><th>Possible values</th></tr><tr><td>`mapbox/driving` </td><td>`driving`, `ferry`, `unaccessible`</td></tr><tr><td>`mapbox/walking`</td><td>`walking`, `ferry`, `unaccessible`</td></tr><tr><td>`mapbox/cycling`</td><td>`cycling`, `walking`, `ferry`, `train`, `unaccessible`</td></tr></table>
`pronunciation` | A string containing an [IPA](https://en.wikipedia.org/wiki/International_Phonetic_Alphabet) phonetic transcription indicating how to pronounce the name in the `name` property. Omitted if pronunciation data is not available for the step.
`intersections` | An array of objects representing all the intersections along the step. See the following table for more information on the `intersections` array:

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


`intersections` | Description
--- | ---
`location` | A `[longitude, latitude]` pair describing the location of the turn.
`bearings` | A list of bearing values that are available at the intersection. The bearings describe all available roads at the intersection.
`classes` | An array of strings signifying the classes of the road exiting the intersection.  <table><tr><th>Possible values</th><th>Description</th></tr><tr><td>`toll`</td><td>Road continues on a toll road</td></tr><tr><td>`ferry`</td><td>Road continues on a ferry</td></tr><tr><td>`restricted`</td><td>Road continues on with access restrictions</td></tr><tr><td>`motorway`</td><td>Road continues on a motorway</td></tr><tr><td>`tunnel`</td><td>Road continues in a tunnel</td></tr></table>
`entry` | A list of entry flags, corresponding 1:1 to `bearings`. If `true`, indicates that the respective road could be entered on a valid route. If `false`, the turn onto the respective road would violate a restriction.
`in` | The index in the `bearings` and `entry` arrays. Used to calculate the bearing before the turn. Namely, the clockwise angle from true north to the direction of travel before the maneuver/passing the intersection. To get the bearing in the direction of driving, the bearing has to be rotated by a value of 180. The value is not supplied for departure maneuvers.
`out` | The index in the `bearings` and `entry` arrays. Used to extract the bearing after the turn. Namely, the clockwise angle from true north to the direction of travel after the maneuver/passing the intersection. The value is not supplied for arrival maneuvers.
`lanes` | An array of [lane](#lane-object) objects that represent the available turn lanes at the intersection. If no lane information is available for an intersection, the `lanes` property will not be present.

`intersections` | 描述
--- | ---
`location` | 一个 `[longitude, latitude]` 描述转弯点的位置。
`bearings` | 一个列表，列出交叉路口可用的方向值，描述交叉路口的所有可用道路。
`classes` | 一组字符串，表示离开交叉路口的道路类别。  <table><tr><th>可能值</th><th>描述</th></tr><tr><td>`toll`</td><td>连接到收费公路</td></tr><tr><td>`ferry`</td><td>连接到轮渡</td></tr><tr><td>`restricted`</td><td>有通行限制</td></tr><tr><td>`motorway`</td><td>连接到公路</td></tr><tr><td>`tunnel`</td><td>连接到隧道</td></tr></table>
`entry` | 一个关于入口标志的列表，1:1 对应 `bearings`. 如果为`true`，则表示可以在有效路线上进入相应的道路，如果为`false`，转向相应的道路将违反限制。
`in` | `bearing`和`entry`数组中的索引。用于计算转弯之前的轴向。即，在操纵/通过交叉点之前从真北到行进方向的顺时针角度。为了使轴朝向行驶方向，轴向必须旋转180°。该值不用于离场行为。
`out` | `bearing`和`entry`数组中的索引。用于提取转弯之后的轴向。即，在操纵/通过交叉点之后从真北到行进方向的顺时针角度。到达行为不提供该值。
`lanes` | 一个[lane](#lane-object) 数组，代表交叉路口的可用转弯车道。如果没有可用于交叉路口的车道信息，则不会出现`lanes`属性。


#### Example route step object
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

### Step maneuver object
### 步骤操作对象

路径步骤对象包含嵌套的**步骤操作**对象，其中包含以下属性：

Property          | Description
------------------| ------------------
`bearing_before`  | A number between `0` and `360` indicating the clockwise angle from true north to the direction of travel immediately _before_ the maneuver.
`bearing_after`   | A number between `0` and `360` indicating the clockwise angle from true north to the direction of travel immediately _after_ the maneuver.
`instruction`     | A human-readable instruction of how to execute the returned maneuver.
`location`        | An array of `[longitude, latitude]` coordinates for the point of the maneuver.
`modifier`        | An optional string indicating the direction change of the maneuver. The meaning of each `modifier` depends on the `type` property. <table><tr><th>Possible values</th><th>Description</th></tr><tr><td>`uturn`</td><td>Indicates a reversal of direction. `type` can be `turn` or `continue` when staying on the same road.</td></tr><tr><td>`sharp right`</td><td>A sharp right turn.</td></tr><tr><td>`right`</td><td>A normal turn to the right.</td></tr><tr><td>`slight right`</td><td>A slight turn to the right.</td></tr><tr><td>`straight`</td><td>No relevant change in direction.</td></tr><tr><td>`slight left`</td><td>A slight turn to the left.</td></tr><tr><td>`left`</td><td>A normal turn to the left.</td></tr><tr><td>`sharp left`</td><td>A sharp turn to the left.</td></tr></table>
`type`            | A string indicating the type of maneuver. See the full list of maneuver types  in the [maneuver types table](#maneuver-types).

- If no `modifier` is provided, the `type` of maneuvers is limited to `depart` and `arrive`.
- If the source or target location is close enough to the `depart` or `arrive` location, no `modifier` will be given.

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

**Maneuver types**<a id='maneuver-types'></a>

`type` | Description
 --- | ---
`turn` | A basic turn in the direction of the modifier.
`new name` | The road name changes (after a mandatory turn).
`depart` | Indicates departure from a leg. The `modifier` value indicates the position of the departure point in relation to the current direction of travel.
`arrive` | Indicates arrival to a destination of a leg. The `modifier` value indicates the position of the arrival point in relation to the current direction of travel.
`merge` | Merge onto a street.
`on ramp` | Take a ramp to enter a highway.
`off ramp` | Take a ramp to exit a highway.
`fork` | Take the left or right side of a fork.
`end of road` | Road ends in a T intersection.
`continue` | Continue on a street after a turn.
`roundabout` | Traverse roundabout. Has an additional property `exit` in the [route step](#routestep-object) that contains the exit number. The `modifier` specifies the direction of entering the roundabout.
`rotary` | A traffic circle. While very similar to a larger version of a roundabout, it does not necessarily follow roundabout rules for right of way. It can offer `rotary_name` parameters, `rotary_pronunciation` parameters, or both, located in the [route step](#routestep-object) object in addition to the `exit` property.
`roundabout turn` | A small roundabout that is treated as an intersection.
`notification` | Indicates a change of driving conditions, for example changing the `mode` from `driving` to `ferry`.
`exit roundabout` | Indicates the exit maneuver from a roundabout. Will not appear in results unless you supply the `roundabout_exits=true` query parameter in the request.
`exit rotary` | Indicates the exit maneuver from a rotary. Will not appear in results unless you supply the `roundabout_exits=true` query parameter in the request.

**Note:** New properties (potentially depending on `type`) may be introduced in the future without an API version change.

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


#### Example step maneuver object
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

### Lane object

A route step object contains a nested **lane object**. The lane object describes the available turn lanes at an intersection. Lanes are provided in their order on the street, from left to right.

Property | Description
--- | ---
 `valid` | Indicates whether a lane can be taken to complete the maneuver (`true`) or not (`false`). For instance, if the lane array has four objects and the first two are valid, the driver can take either of the left lanes and stay on the route.
 `indications` | An array of signs for each turn lane. There can be multiple signs. For example, a turn lane can have a sign with an arrow pointing left and another sign with an arrow pointing straight.

#### Example lane object

```json
{
    "valid": true,
    "indications": [
        "left"
    ]
}
```

### Voice instruction object

A route step object contains a nested **voice instruction object** if the optional `voice_instructions=true` query parameter is present. The voice instruction object contains the text that should be announced, along with how far from the maneuver it should be emitted. The voice instructions are children of the route step during which they should be spoken, but they refer to the maneuver in the _following_ step.

Property | Description
--- | ---
`distanceAlongGeometry` | A float indicating how far from the upcoming maneuver the voice instruction should begin in meters.
`announcement` | A string containing the text of the verbal instruction.
`ssmlAnnouncement` | A string with SSML markup for proper text and pronunciation. This property is designed for use with [Amazon Polly](https://aws.amazon.com/polly/). The SSML tags may not work with other text-to-speech engines.

#### Example voice instruction object

```json
{
    "distanceAlongGeometry": 375.7,
    "announcement": "In a quarter mile, take the ramp on the right towards I 495: Baltimore",
    "ssmlAnnouncement": "<speak><amazon:effect name=\"drc\"><prosody rate=\"1.08\">In a quarter mile, take the ramp on the right towards <say-as interpret-as=\"address\">I-495</say-as>: Baltimore</prosody></amazon:effect></speak>"
}
```

### Banner instruction object

A route step object contains a nested **banner instruction object** if the optional `banner_instructions=true` query parameter is present. The banner instruction object contains the contents of a banner that should be displayed as added visual guidance for a route. The banner instructions are children of the route steps during which they should be displayed, but they refer to the maneuver in the _following_ step.

Property | Description
--- | ---
`distanceAlongGeometry` | A float indicating how far from the upcoming maneuver the banner instruction should begin being displayed in meters. Only one banner should be displayed at a time.
`primary` | The most important content to display to the user. This text is larger and at the top.
`secondary` | Additional content useful for visual guidance. This text is slightly smaller and below `primary`. Can be `null`.
`sub` | Additional information that is included if the driver needs to be notified about something. Can include information about the _next_ maneuver (the one after the upcoming one) if the step is short. If lane information is available, that takes precedence over information about the _next_ maneuver.

Each of the different banner types (`primary`, `secondary`, and `sub`) contains the following properties:

Property | Description
--- | ---
`text` | A string that contains all the text that should be displayed.
`type` | The type of maneuver. May be used in combination with the `modifier` (and, if it is a roundabout, the `degrees`) to for an icon to display. Possible values: `turn`, `merge`, `depart`, `arrive`, `fork`, `off ramp`, and `roundabout`.
`modifier`<br>(optional) | The modifier for the maneuver. Can be used in combination with the `type` (and, if it is a roundabout, the `degrees`) to for an icon to display. <table><tr><th>Possible values</th><th>Description</th></tr><tr><td>`uturn`</td><td>Indicates a reversal of direction</td></tr><tr><td>`sharp right`</td><td>A sharp right turn</td></tr><tr><td>`right`</td><td>A normal turn to the right</td></tr><tr><td>`slight right`</td><td>A slight turn to the right</td></tr><tr><td>`straight`</td><td>No relevant change in direction</td></tr><tr><td>`slight left`</td><td>A slight turn to the left</td></tr><tr><td>`left`</td><td>A normal turn to the left</td></tr><tr><td>`sharp left`</td><td>A sharp turn to the left</td></tr></table>
`degrees`<br>(optional) | The degrees at which you will be exiting a roundabout, assuming `180` indicates going straight through the roundabout.
`driving_side`<br>(optional) | A string representing which side the of the street people drive on in that location. Can be `left` or `right`.
`components` | Objects that, together, make up what should be displayed in the banner. Includes additional information intended to be used to aid in visual layout. A component can contain the following properties:

`components` | Description
--- | ---
`type` | A string with more context about the component that may help in visual markup and display choices. If the type of the component is unknown, it should be treated as text. <table><tr><th>Possible values</th><th>Description</th></tr><tr><td>`text`</td><td>Default. Indicates the text is part of the instructions and no other type.</td></tr><tr><td>`icon` </td><td>This is text that can be replaced by an imageBaseURL icon.</td></tr><tr><td>`delimiter`</td><td>This is text that can be dropped, and should be dropped if you are rendering icons.</td></tr><tr><td>`exit-number`</td><td>Indicates the exit number for the maneuver.</td></tr><tr><td>`exit`</td><td>Provides the the word for _exit_ in the local language.</td></tr><tr><td>`lane`</td><td>Indicates which lanes can be used to complete the maneuver.</td></tr></table> The introduction of new types is not considered a breaking change.
`text` | The sub-string of the `text` of the parent objects that may have additional context associated with it
`abbr`<br>(optional) | The abbreviated form of `text`. If this is present, there will also be an `abbr_priority` value. For an example of using `abbr` and `abbr_priority`, see the [abbreviation examples](#abbreviation-examples) table.
`abbr_priority`<br>(optional) | An integer indicating the order in which the abbreviation `abbr` should be used in place of `text`. The highest priority is `0`, while a higher integer value means it should have a lower priority. There are no gaps in integer values. Multiple components can have the same `abbr_priority`. When this happens, all `components` with the same `abbr_priority` should be abbreviated at the same time. Finding no larger values of `abbr_priority` means that the string is fully abbreviated.
`imageBaseURL`<br>(optional) | A string pointing to a shield image to use instead of the text.
`directions` | An array indicating which directions you can go from a lane (left, right, or straight). If the value is ['left', 'straight'], the driver can go straight or left from that lane. Present if `components.type` is `lane`.
`active` | A boolean that tells you whether that lane can be used to complete the upcoming maneuver. If multiple lanes are active, then they can all be used to complete the upcoming maneuver. Present if `components.type` is `lane`.

**Abbreviation examples**<a id='abbreviation-examples'></a>

`text`             | `abbr`       | `abbr_priority`
-------------------|--------------|--------------------
North              | N            | 1
Franklin Drive     | Franklin Dr  | 0

Given the `components` in the table above, the possible abbreviations are, in order:
- North Franklin Dr
- N Franklin Dr

#### Example banner instruction object

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

### Directions API errors

On error, the server responds with different HTTP status codes. For responses with HTTP status codes lower than `500`, the JSON response body includes the `code` property, which may be used by client programs to manage control flow. The response body may also include a `message` property with a human-readable explanation of the error.

If a server error occurs, the HTTP status code will be `500` or higher and the response will not include a `code` property.

Response body `code` | HTTP status code | Description
--- | --- |---
`Ok` | `200` | Normal success case.
`NoRoute` | `200` | There was no route found for the given coordinates. Check for impossible routes (for example, routes over oceans without ferry connections).
`NoSegment` | `200` | No road segment could be matched for coordinates. Check for coordinates that are too far away from a road.
`ProfileNotFound` | `404` | Use a valid profile as described in the [list of routing profiles](#directions).
`InvalidInput` | `422` | The given request was not valid. The `message` key of the response will hold an explanation of the invalid input.
