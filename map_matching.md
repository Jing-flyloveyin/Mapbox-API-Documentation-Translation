## Map Matching

这份文件是Map Matching API 的版本5 “`v5`”. 如要了解先前版本信息,见[`v4` documentation](./pages/map_matching_v4.html).

Mapbox Map Matching API 利用Directions API 将在GPS装置或手机上捕捉到的不清楚或不准确的痕迹发送到OpenStreetMap路网，然后产生可用于地图或其他分析的清晰道路。Map Matching API还能够根据疑问回应详细说明。

Map Matching 的 Swift and Objective-C 支持由 [MapboxDirections.swift](https://github.com/mapbox/MapboxDirections.swift)
library提供.

```python
from mapbox import MapMatcher
```

```javascript
const mbxMapMatching = require('@mapbox/mapbox-sdk/services/mapMatching');
const mapMatchingClient = mbxMapMatching({ accessToken: '{your_access_token}' });
```

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

**限制**

- Map Matching API 每分钟最多只能回应60个请求。
- 每个请求最多只能包含100个坐标。
- 必须使用Mapbox [libraries or SDKs](https://mapbox.com/developers)开发成果并显示在Mapbox地图上。

如要了解大量数据操作或其他使用情况，请联系我们 （[contact us](https://mapbox.com/contact/)）。

### 寻回匹配

```endpoint
GET /matching/v5/{profile}/{coordinates}.json
```

在路网中寻回与输入痕迹最接近的道路。

URL 参数 | 描述
--- | ---
`profile` | Mapbox Directions路由文件ID <table><tr><th>Profile ID</th><th>Description</th></tr><tr><td>`mapbox/driving`</td><td>行车时间、距离或两者皆包括</td></tr><tr><td>`mapbox/walking`</td><td>步行时间/距离或两者皆包括</td></tr><tr><td>`mapbox/cycling`</td><td>骑车时间/距离或两者皆包括</td></tr><tr><td>`mapbox/driving-traffic`</td><td>交通数据报告的行车时间/距离或两者皆包括</td></tr></table>
`coordinates` | 可有序访问的一系列分号分隔的 `{longitude},{latitude}`坐标。 可包含`2`到`100`个坐标。

你可以使用下列自选参数继续改善目前为止产生的成果:

查询参数 | 描述
--- | ---
`annotations`<br /> (可选择) | 返回路径元数据. 可能的值有: `duration`（持续时间）, `distance`（距离）, 和 `speed`（速度）. 可将数个注释，包含在一份逗号分隔的列表中。 关于注释内的更多细节，见 [route leg object](#routeleg-object)。
`approaches`<br /> (可选择) | 一份分号分隔的列表，表明要求路线中到达路径点的道路边线。接受 `unrestricted` (默认, 路线可从任意路边到达路径点) 或者 `curb` (路线会从区域的行驶边线`driving_side`到达路径点). 若提供途径approaches的数据，必须和路径点的数量相同。 但是，你可以跳过一个坐标并用`;`分号表示其位置。必须和`steps=true`一起使用。
`geometries`<br /> (可选择) | 返回几何的形式。允许的值有: `geojson` (as [LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2)), [`polyline`](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) (默认, 精准度为5的多段线), 和 [`polyline6`](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) (精准度为6的多段线).
`language`<br /> (可选择) | 返回的路线规划文字指令的语言。 详见 [supported languages](#instructions-languages). 默认语言是英语 `en` (English)。 必须和 `steps=true`一起使用.
`overview`<br /> (可选择) | 返回的全览几何的类型. 可以是`full` (最详细的可用几何), `simplified` (默认, 简化的全览几何), or `false` (无全览几何)。
`radiuses`<br /> (可选择) | 一个坐标可以用来对齐路网的最大移动距离，以米为单位。 Radiuses的数量必须和要求中坐标数量相同并以`;`分隔。值可以是`0.0` 到 `50.00`之间的任意数. 在痕迹杂乱时，用较大的数字(`20`-`50`)；在痕迹干净时，用较小的数字(`1`-`10`)。 默认值为`5`。
`steps`<br /> (可选择) | 是否返回steps或路线规划指令。是 (`true`) 或否 (`false`, 默认)。
`tidy`<br /> (可选择) | 是否删除因改善地图匹配结果而产生的群集和重新取样痕迹。是 (`true`)或否(`false`, 默认).
`timestamps`<br /> (可选择) | 数据以[Unix time](https://en.wikipedia.org/wiki/Unix_time)的格式呈现并与请求中每个坐标关联。时间戳的数量必须和请求的坐标数量相同。同时，用升序排列并用分号`;`分隔。为产生最好的结果，时间戳的取样频率应约为5。
`waypoint_names`<br /> (可选择) | 一份分号分隔的坐标自定义命名列表，用于横幅和语音指令中的到达指令。值可以是任意字符串，但字符的总数量不能超过500。 `waypoint_names`的列表长度必须和坐标列表长度相同，并以分号分隔。但是，你可以跳过一个坐标并以分号`;`代替它的位置。
`waypoints`<br /> (可选择) | 表明该输入坐标应被当作路径点。所有在[match object's](#match-object)路线中有到达和出发时间的坐标都会被默认为路径点. 指数可以与任意输入坐标相关，但是必须包含第一个(`0`)和最后一个坐标，并以`;`分隔。与`steps=true`一起使用，并以高取样频率为根据请求将产生最有效的结果。

如果匹配成功，只有一个匹配对象会被返回。当算法无法决定两个点中的正确匹配，该线将会被忽略，同时多个次匹配对象将会被返回。次匹配对象的数量越多，越表明输入痕迹与路网的不良匹配。

预处理的技巧:

- 非常推荐时间戳，因为可以提高匹配的质量。
- Map Matching API 能处理的痕迹最多只能包括100个坐标。如果需要处理更长的痕迹，可以分离该痕迹并请求多次。
- 多个点（例如一个人在火车轨道路口等了几分钟）通常不会给痕迹添加信息，并且会负面地影响匹配质量。 因此，建议清理痕迹（清除群集并提供统一的取样频率）。 你可以使用`tidy=true`查询参数，或者使用类似[geojson-tidy](https://github.com/mapbox/geojson-tidy)的外部工具预处理你的痕迹。
- 当两点间的取样频率在5时，将会产生最好的地图匹配成果。如果你的采样频率高于5，建议降低痕迹的采样频率。
- 当`waypoints`参数被指定时，痕迹通常会返回错误的次匹配对象。 因此，我们建议在使用`waypoints`参数前清理痕迹。  

#### 请求的例子

```curl
# 基本的请求会返回路径点之间的匹配对象
curl "https://api.mapbox.com/matching/v5/mapbox/driving/-117.17282,32.71204;-117.17288,32.71225;-117.17293,32.71244;-117.17292,32.71256;-117.17298,32.712603;-117.17314,32.71259;-117.17334,32.71254?access_token={your_access_token}"

# 包含每个路径点到路边的处理参数
curl "https://api.mapbox.com/matching/v5/mapbox/driving/-117.17282,32.71204;-117.17288,32.71225;-117.17293,32.71244;-117.17292,32.71256?approaches=curb;curb;curb;curb&access_token={your_access_token}"


# 包含各种参数的请求，会返回在第一个和最后一个路径点之间的单一匹配对象，且只有一段路线段
curl "https://api.mapbox.com/matching/v5/mapbox/driving/2.344003,48.85805;2.34675,48.85727;2.34868,48.85936;2.34955,48.86084;2.34955,48.86088;2.34962,48.86102;2.34982,48.86125?steps=true&tidy=true&waypoints=0;6&waypoint_names=Home;Work&banner_instructions=true&access_token={your_access_token}"
```

```python
service = MapMatcher()
#输入数据必须是一个带有LineString 几何的 GeoJSON-like Feature
# 可自选的coordTimes的性质必须是包含时间戳的数组
line = {
    "type": "Feature",
    "properties": {
        "coordTimes": [
            "2015-04-21T06:00:00Z",
            "2015-04-21T06:00:05Z",
            "2015-04-21T06:00:10Z",
            "2015-04-21T06:00:15Z",
            "2015-04-21T06:00:20Z"]},
    "geometry": {
        "type": "LineString",
        "coordinates": [
            [13.418946862220764, 52.50055852688439],
            [13.419011235237122, 52.50113000479732],
            [13.419756889343262, 52.50171780290061],
            [13.419885635375975, 52.50237416816131],
            [13.420631289482117, 52.50294888790448]]}}
response = service.match(line, profile='mapbox.driving')
# 回应 geojson应包含一个只有单一特征的FeatureCollection
# 且包含新矫正的LineString来匹配备选的文件段落
corrected = response.geojson()['features'][0]
corrected['geometry']['type']
'LineString'
corrected['geometry'] == line['geometry']
>>> False
len(corrected['geometry']) == len(line['geometry'])
>>> True
```

```javascript
mapMatchingClient
  .getMatching({
    points: [
      {
        coordinates: [-117.17283, 32.712041],
        approach: 'curb'
      },
      {
        coordinates: [-117.17291, 32.712256],
        isWaypoint: false
      },
      {
        coordinates: [-117.17292, 32.712444]
      },
      {
        coordinates: [-117.172922, 32.71257],
        waypointName: 'point-a',
        approach: 'unrestricted'
      },
      {
        coordinates: [-117.172985, 32.7126],
        timestamp: '2018-06-08T10:48:19.307Z'
      },
      {
        coordinates: [-117.173143, 32.712597]
      },
      {
        coordinates: [-117.173345, 32.712546]
      }
    ],
    tidy: false,
  })
  .send()
  .then(response => {
    const matching = response.body;
  })
```

```bash
// 尚不支持。
```

```java
List<Point> coordinates = new ArrayList<>();
coordinates.add(Point.fromLngLat(-117.17283, 32.712041);
coordinates.add(Point.fromLngLat(-117.17291, 32.712256);
coordinates.add(Point.fromLngLat(-117.17292, 32.712444);
coordinates.add(Point.fromLngLat(-117.172922, 32.71257);
coordinates.add(Point.fromLngLat(-117.172985, 32.7126);
coordinates.add(Point.fromLngLat(-117.173143, 32.712597);
coordinates.add(Point.fromLngLat(-117.173345, 32.712546);

MapboxMapMatching client = MapboxMapMatching.builder()
  .accessToken("{your_access_token}")
  .profile(DirectionsCriteria.PROFILE_DRIVING)
  .coordinates(coordinates)
  .build();
  client.enqueueCall(new Callback<MapMatchingResponse>() {
  @Override
  public void onResponse(Call<MapMatchingResponse> call, Response<MapMatchingResponse> response) {
	  if (response.isSuccessful()) {
	        // Work with response
    } else {
        // If the response code does not response "OK" an error has occured
    }
  }

  @Override
  public void onFailure(Call<MapMatchingResponse> call, Throwable throwable) {
		// Deal with failure here

  }
});  
```

```objc
NSArray<MBWaypoint *> *waypoints = @[
    [[MBWaypoint alloc] initWithCoordinate:CLLocationCoordinate2DMake(32.712041, -117.172836) coordinateAccuracy:-1 name:nil],
    [[MBWaypoint alloc] initWithCoordinate:CLLocationCoordinate2DMake(32.712256, -117.17291) coordinateAccuracy:-1 name:nil],
    [[MBWaypoint alloc] initWithCoordinate:CLLocationCoordinate2DMake(32.712444, -117.17292) coordinateAccuracy:-1 name:nil],
    [[MBWaypoint alloc] initWithCoordinate:CLLocationCoordinate2DMake(32.71257, -117.172922) coordinateAccuracy:-1 name:nil],
    [[MBWaypoint alloc] initWithCoordinate:CLLocationCoordinate2DMake(32.7126, -117.172985) coordinateAccuracy:-1 name:nil],
    [[MBWaypoint alloc] initWithCoordinate:CLLocationCoordinate2DMake(32.712597, -117.173143) coordinateAccuracy:-1 name:nil],
    [[MBWaypoint alloc] initWithCoordinate:CLLocationCoordinate2DMake(32.712546, -117.173345) coordinateAccuracy:-1 name:nil],
];

MBMatchOptions *matchOptions = [[MBMatchOptions alloc] initWithWaypoints:waypoints profileIdentifier:MBDirectionsProfileIdentifierAutomobile];
NSURLSessionDataTask *task = [[[MBDirections alloc] initWithAccessToken:MapboxAccessToken] calculateMatchesWithOptions:matchOptions completionHandler:^(NSArray<MBMatch *> * _Nullable matches, NSError * _Nullable error) {
    if (error) {
        NSLog(@"Error matching locations: %@", error);
        return;
    }

    MBMatch *match = matches.firstObject;
    MBRouteLeg *leg = match.legs.firstObject;
    if (leg) {
        NSLog(@"Match via %@:", leg);

        NSLengthFormatter *distanceFormatter = [[NSLengthFormatter alloc] init];
        NSString *formattedDistance = [distanceFormatter stringFromMeters:leg.distance];

        NSDateComponentsFormatter *travelTimeFormatter = [[NSDateComponentsFormatter alloc] init];
        travelTimeFormatter.unitsStyle = NSDateComponentsFormatterUnitsStyleShort;
        NSString *formattedTravelTime = [travelTimeFormatter stringFromTimeInterval:match.expectedTravelTime];

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
let locations = [
    CLLocationCoordinate2D(latitude: 32.712041, longitude: -117.172836),
    CLLocationCoordinate2D(latitude: 32.712256, longitude: -117.17291),
    CLLocationCoordinate2D(latitude: 32.712444, longitude: -117.17292),
    CLLocationCoordinate2D(latitude: 32.71257,  longitude: -117.172922),
    CLLocationCoordinate2D(latitude: 32.7126,   longitude: -117.172985),
    CLLocationCoordinate2D(latitude: 32.712597, longitude: -117.173143),
    CLLocationCoordinate2D(latitude: 32.712546, longitude: -117.173345)
]

let options = MatchOptions(coordinates: locations)
options.includesSteps = true

let task = directions.calculate(options) { (matches, error) in
    guard error == nil else {
        print("Error matching locations: \(error!)")
        return
    }

    if let match = matches?.first, let leg = match.legs.first {
        print("Match via \(leg):")

        let distanceFormatter = LengthFormatter()
        let formattedDistance = distanceFormatter.string(fromMeters: match.distance)

        let travelTimeFormatter = DateComponentsFormatter()
        travelTimeFormatter.unitsStyle = .short
        let formattedTravelTime = travelTimeFormatter.string(from: match.expectedTravelTime)

        print("Distance: \(formattedDistance); ETA: \(formattedTravelTime!)")

        for step in leg.steps {
            print("\(step.instructions)")
            let formattedDistance = distanceFormatter.string(fromMeters: step.distance)
            print("— \(formattedDistance) —")
        }
    }
}
```

#### 回应的例子 The Example of Response
```json
{
  "matchings": [
    {
      "confidence": 0.816975633159885,
      "geometry": "{qeiHayhMzCsQLW@_@Ia@UY{TaK",
      "legs": [
        {
          "summary": "Quai de la Mégisserie, Boulevard de Sébastopol",
          "weight": 194.8,
          "duration": 143.7,
          "steps": [ ],
          "distance": 701.9
        }
      ],
      "weight_name": "routability",
      "weight": 194.8,
      "duration": 143.7,
      "distance": 701.9
    }
  ],
  "tracepoints": [ ],
  "code": "Ok"
}
```

### 匹配回应对象 Match Response Object

**match response object** **匹配回应对象**包含一个或多个匹配对象[match objects](#match-object), 以及一个或多个跟踪点对象 [tracepoint objects](#tracepoint-object)。

性质 | 描述
--- | ---
`code` | 一个表明回应状态的字符串。 可能的值已被列在[Map Matching status codes section](#map-matching-status-codes)中。
`matchings` | [match objects](#match-object)的数组.
`tracepoints` | [tracepoint objects](#tracepoint-object)数组，代表输入点所匹配的位置，按匹配顺序排列。 如果一个跟踪点被Map Matching API忽略，它将显示`null`。

如果匹配成功，只有一个匹配对象会被返回。当算法无法决定两个点中的正确匹配，该线将会被忽略，同时多个次匹配对象将会被返回。次匹配对象的数量越多，越表明输入痕迹与路网的不良匹配。

#### 匹配回应对象的例子 The Example of Match Response Object

```json
{
  "matchings": [
    {
      "confidence": 0.9857403441039709,
      "geometry": "gatfEfidjUk@Lc@@Y?",
      "legs": [
        {
          "summary": "",
          "weight": 2.7,
          "duration": 2.7,
          "steps": [],
          "distance": 25
        },
        {
          "summary": "",
          "weight": 2.4,
          "duration": 2.4,
          "steps": [],
          "distance": 21
        },
        {
          "summary": "",
          "weight": 1.6,
          "duration": 1.6,
          "steps": [],
          "distance": 14
        }
      ],
      "weight_name": "routability",
      "weight": 6.699999999999999,
      "duration": 6.699999999999999,
      "distance": 60
    }
  ],
  "tracepoints": [
    {
      "alternatives_count": 0,
      "waypoint_index": 0,
      "matchings_index": 0,
      "name": "North Harbor Drive",
      "location": [
        -117.172836,
        32.712041
      ]
    },
    {
      "alternatives_count": 0,
      "waypoint_index": 1,
      "matchings_index": 0,
      "name": "North Harbor Drive",
      "location": [
        -117.17291,
        32.712256
      ]
    },
    {
      "alternatives_count": 0,
      "waypoint_index": 2,
      "matchings_index": 0,
      "name": "North Harbor Drive",
      "location": [
        -117.17292,
        32.712444
      ]
    },
    {
      "alternatives_count": 9,
      "waypoint_index": 3,
      "matchings_index": 0,
      "name": "North Harbor Drive",
      "location": [
        -117.172922,
        32.71257
      ]
    }
  ],
  "code": "Ok"
}
```

### 匹配对象 （The Match Object）

**match object** **匹配对象**是包含额外置信度的[route object](#route-object）:

性质 | 描述
--- | ---
`confidence` | 表明返回匹配置信度的浮点数，从`0` (低) 到 `1` (高)。
`distance` | 表明旅行距离的浮点数，以米为单位。
`duration` | 表明估计的旅行时间的浮点数，以秒为单位。
`weight` | 表明重量的字符串，单位与`weight_name`中所用相同.
`weight_name` | 表明所用重量的字符串. 默认值为`routability`, 指给予不理想策略，基于持续时间的额外处罚。
`geometry` | 由请求中的`geometries`参数决定时, 是[GeoJSON LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2)或者[Polyline string](https://developers.google.com/maps/documentation/utilities/polylinealgorithm).由请求中的`overview`参数决定时, 是一个完整的路线几何(`full`),或者是一个在完整几何可见的缩放级别时，被简化的几何(`simplified`),抑或者是不包括(`false`)。
`legs` | [route leg objects](#routeleg-object)的数组。
`voice_locale` | 语音指令的区域设置。默认为英语`en` (English). 详见 [supported languages](#instructions-languages).可自选，当 `steps=true`时，会被包含。

#### 匹配对象的例子 Example match object

```json
{
    "confidence": 0.9548844020537051,
    "distance": 103.7,
    "duration": 16.4,
    "geometry": "gatfEfidjUi@Le@@Y?E??J?^Hf@",
    "legs": []
}
```

### 跟踪点对象 The tracepoint object

 **tracepoint object** **跟踪点对象**是额外包含`matchings_index`, `waypoint_index`, 和 `alternatives_count` 三个域的路径点对象 [waypoint object](#waypoint-object)。

性质 | 描述
--- | ---
`matchings_index` | 在`matchings`中，次痕迹所匹配的匹配对象的指数。
`waypoint_index` | 匹配路径中所包含的路径点的指数。
`alternatives_count` | 该跟踪点可替换的匹配的数量。当值为`0`时，表明该点是被绝对匹配的。适合在这样的点上分离痕迹，以达到增量的地图匹配。
`name` | 坐标所对齐的道路名称。
`location` | 包含对齐坐标位置的数组，以经纬度`[longitude, latitude]`的格式呈现。

#### 跟踪点的例子 Example tracepoint object

```json
{
    "waypoint_index": 0,
    "location": [-117.172836, 32.71204],
    "name": "North Harbor Drive",
    "matchings_index": 0,
    "alternatives_count": 0
}
```

### 地图匹配状况的代码 Map Matching status codes

当发生错误时, 服务器会回应不同的HTTP状态代码。

- 当回应的HTTP状态代码小于`500`时,JSON回应主体会包括代码性质，客户程序可以用其来管理控制流。回应主体可能会同时包括信息性质, 是所发生的错误的可读解释。
- 当服务器发生错误时, HTTP状态代码将会呈 `500` 或更高，同时回应将不会包括代码`code`的性质。

状态代码 | 描述
--- | ---
`Ok` | 正常
`NoMatch` | 输入数据没有产生任何匹配，或者请求的`waypoints`没有在匹配成果中找到。`features`将会是空的数组。
`TooManyCoordinates` | 请求中的点数量大于100。
`InvalidInput` | `message`将持有无效输入的解释。
`ProfileNotFound` | 文件配置必须是有效的 (`mapbox/driving`,  `mapbox/driving-traffic`, `mapbox/walking`, 或  `mapbox/cycling`)。

### 使用HTTP POST Using HTTP POST

Map Matching API通过HTTP `POST`的方式支持. 若要使用HTTP `POST`提交请求，必须遵循以下所言改变请求：

  1. HTTP的方式必须是`POST`
  2. 请求的`Content-Type`必须是`application/x-www-form-urlencoded`
  3. 坐标列表必须在请求主体中以`coordinates=` 参数的方式呈现, 同时URL中不可包含任何坐标
  4. `access_token`参数不可包含在主体内容中，而应置于 `POST` URL中
  5. 所有其他的参数必须包含在请求主体内容中

`POST`请求的例子如下：

```
POST /matching/v5/mapbox/driving?access_token={your_access_token} HTTP/1.0
Content-Type: application/x-www-form-urlencoded

coordinates=2.344003915786743,48.85805170891599;2.346750497817993,48.85727523615161;2.348681688308716,48.85936462637049;2.349550724029541,48.86084691113991;2.349550724029541,48.8608892614883;2.349625825881958,48.86102337068847;2.34982967376709,48.86125629633996&steps=true&tidy=true&waypoints=0;6
```

```curl
# 在POST界面上的基本请求
$ curl -d "coordinates=-117.17282,32.71204;-117.17288,32.71225;-117.17293,32.71244;-117.17292,32.71256;-117.17298,32.712603;-117.17314,32.71259;-117.17334,32.71254"
"https://api.mapbox.com/matching/v5/mapbox/driving?access_token={your_access_token}"

# 包含各种参数的请求，会返回在第一个和最后一个路径点之间的单一匹配对象，且只有一段路线段。
$ curl -d "coordinates=2.344003,48.85805;2.34675,48.85727;2.34868,48.85936;2.34955,48.86084;2.34955,48.86088;2.349625,48.86102;2.34982,48.86125&steps=true&tidy=true&waypoints=0;6&waypoint_names=Home;Work&banner_instructions=true" "https://api.mapbox.com/matching/v5/mapbox/driving?access_token={your_access_token}"
```
