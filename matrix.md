## Matrix

Matrix API 返回多点之间的移动时间信息。

例如, 给出3个位置 A， B，和C， Matrix API 将返回这些位置之间以秒为单位的时间信息矩阵:

| |   A   |   B   |   C
--| ----- | ----- | -----
A | A → A | A → B | A → C
B | B → A | B → B | B → C
C | C → A | C → B | C → C

Matrix API将始终返回每个元素的持续时间或最快的路径距离，其中元素是矩阵中的原点-目标点。 Matrix API会返回以秒为单位的持续时间和以米为单位的距离信息。不会返回路径的几何信息。

点与点之间的时间或距离信息可能不是对称的，因为路线可能因单行道或转弯限制而方向不同。例如，A到B可能不同于B到A的持续时间。

Matrix API允许您有效地检查彼此坐标的可达性，根据移动时间过滤点，或者运行您自己的算法来解决问题并优化。

**约束与限制**
- For the `mapbox/driving`, `mapbox/walking`, and `mapbox/cycling` profiles:
    - 每次请求最多输入25个坐标
    - 每分钟最多60次请求
- For the `mapbox/driving-traffic` profile:   
    - 每次请求最多输入10个坐标
    - 每分钟最多30次请求

获取更多信息, [联系我们](https://www.mapbox.com/contact/).

```python
from mapbox import DirectionsMatrix
```

```javascript
const mbxMatrix = require('@mapbox/mapbox-sdk/services/matrix');
const matrixClient = mbxMatrix({ accessToken: '{your_access_token}' });
```

### Retrieve a matrix
```endpoint
GET /directions-matrix/v1/{profile}/{coordinates}
```

返回一个时间矩阵，或一个距离矩阵，或两者都返回，显示坐标之间移动时间和距离信息。在默认情况下，输入坐标作为源点和目标点，此服务端点返回一个对称矩阵。使用`sources` and `destination`作为可选参数，您还可以使用一些源点和目标点坐标生成非对称矩阵。

URL 参数 | 描述
--- | ---
`profile` | 一个Mapbox Directions路由配置ID. <table><tr><th>Profile ID</th><th>描述</th></tr><tr><td>`mapbox/driving`</td><td>汽车移动时间，距离。或两者都包含。</td></tr><tr><td>`mapbox/walking`</td><td>步行移动时间，距离。或两者都包含</td></tr><tr><td>`mapbox/cycling`</td><td>自行车移动时间，距离。或两者都包含</td></tr><tr><td>`mapbox/driving-traffic`</td><td>交通数据展示的汽车移动时间，距离。或两者都包含。</td></tr></table>
`coordinates` | 一个分号分隔的`{longitude},{latitude}`坐标串。必须有2到25个坐标。

您可以通过以下可选参数进一步细化来自该服务端的结果：

Query 参数 | 描述
--- | ---
`annotations`<br /> (可选) | 用于指定生成的矩阵。可能的值是：`duration` （默认值），`distance`，或者是两个由逗号分隔的值。
`approaches`<br /> (可选) | 一个分号分隔的列表，表示请求的路线中最接近航点的道路。接受`unrestricted`(默认情况下，路线可以从道路两侧到达航点)或`curb`(路线将到达该区域的`driving_side`的航点)。如果提供的话，方法的数量必须与航点的数量相同。但是，您可以略过坐标并用`;` 分隔符显示列表中的位置。
`destinations`<br /> (可选) | 使用给定索引的坐标作为目的地。可能的值是：分号分隔基于0的索引列表，或`all`（默认值）。选项`all`允许使用所有坐标作为目的地。
`sources`<br /> (可选) | 使用给定索引的坐标作为源地。可能的值是：分号分隔基于0的索引列表，或`all`（默认值）。选项`all`允许使用所有坐标作为源地。

查询字符串中的未识别选项将导致`InvalidInput`错误。

#### Example requests

```curl
# Request a symmetric 3x3 matrix for cars with a curbside approach for each destination
curl "https://api.mapbox.com/directions-matrix/v1/mapbox/driving/-122.42,37.78;-122.45,37.91;-122.48,37.73?approaches=curb;curb;curb&access_token={your_access_token}"

# Request an asymmetric 2x3 matrix for bicycles
curl "https://api.mapbox.com/directions-matrix/v1/mapbox/cycling/-122.42,37.78;-122.45,37.91;-122.48,37.73?sources=0;2&destinations=all&access_token={your_access_token}"

# Request a 1x3 matrix for walking that includes both duration and distance
curl "https://api.mapbox.com/directions-matrix/v1/mapbox/walking/-122.418563,37.751659;-122.422969,37.75529;-122.426904,37.759617?sources=1&annotations=distance,duration&access_token={your_access_token}"
```

```python
service = DirectionsMatrix()
# input waypoints are features, typically GeoJSON-like feature dictionaries
portland = {
    'type': 'Feature',
    'properties': {'name': 'Portland, OR'},
    'geometry': {
        'type': 'Point',
        'coordinates': [-122.7282, 45.5801]}}
bend = {
    'type': 'Feature',
    'properties': {'name': 'Bend, OR'},
    'geometry': {
        'type': 'Point',
        'coordinates': [-121.3153, 44.0582]}}
# Matrix method can be called with a list of point features and the travel profile
response = service.matrix([portland, bend], profile='mapbox/driving')
```

```javascript
matrixClient
  .getMatrix({
    points: [
      {
        coordinates: [2.2, 1.1]
      },
      {
        coordinates: [2.2, 1.1],
        approach: 'curb'
      },
      {
        coordinates: [3.2, 1.1]
      },
      {
        coordinates: [4.2, 1.1]
      }
    ],
    profile: 'walking'
  })
  .send()
  .then(response => {
      const matrix = response.body;
  });
```

```bash
# This API is not available through the Mapbox CLI
```

```java
MapboxMatrix directionsMatrixClient = MapboxMatrix.builder()
  .accessToken("{your_access_token}")
  .profile(DirectionsCriteria.PROFILE_DRIVING)
  .coordinates(listOfPoints)
  .build();
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
    "code": "Ok",
    "durations": [
        [ 0,      573, 1169.5 ],
        [ 573,    0,      597 ],
        [ 1169.5, 597,      0 ]
    ],
    "destinations": [
        {
            "name": "Mission Street",
            "location": [ -122.418408, 37.751668 ]
        },
        {
            "name": "22nd Street",
            "location": [ -122.422959, 37.755184 ]
        },
        {
            "name": "",
            "location": [ -122.426911, 37.759695 ]
        }
    ],
    "sources": [
        {
            "name": "Mission Street",
            "location": [ -122.418408, 37.751668 ]
        },
        {
            "name": "22nd Street",
            "location": [ -122.422959, 37.755184 ]
        },
        {
            "name": "",
            "location": [ -122.426911, 37.759695 ]
        }
    ]
}
```

### The matrix response object

The response to a Matrix API request is a JSON object that contains the following properties:

Property | Description
--- | ---
`code` | A string indicating the state of the response. This is a separate code than the HTTP status code. On normal valid responses, the value will be `Ok`. See the errors section below for more information.
`durations` | Durations as an array of arrays that represent the matrix in row-major order. `durations[i][j]` gives the travel time from the i<sup>th</sup> source to the j<sup>th</sup> destination. All values are in seconds. The duration between the same coordinate is always `0`. If a duration cannot be found, the result is `null`.
`distances` | Distances as an array of arrays that represent the matrix in row-major order. `distances[i][j]` gives the travel distance from the i<sup>th</sup> source to the j<sup>th</sup> destination. All values are in meters. The distance between the same coordinate is always `0`. If a distance cannot be found, the result is `null`.
`destinations:` | An array of `waypoint` objects. Each waypoint is an input coordinate snapped to the road and path network. The waypoints appear in the array in the order of the input coordinates, or in the order specified in the `destinations` query parameter.
`sources` | An array of `waypoint` objects. Each waypoint is an input coordinate snapped to the road and path network. The waypoints appear in the array in the order of the input coordinates, or in the order specified in the `sources` query parameter.

**Note:** If no route is found between a source and a destination, the respective value in the `durations` or `distances` matrix will be `null`.

#### Example matrix response object

```json
{
    "code": "Ok",
    "durations": [
        [ 0,    77.3, null ],
        [ 75.7, 0,    null ],
        [ null, null, 0    ]
    ],
    "distances": [
        [ 0,   846.2, null ],
        [ 832.2,   0, null ],
        [ null, null,    0 ]
    ],
    "destinations": [
        {
            "location": [ -6.80897, 62.000075 ],
            "name": "Kirkjubøarvegur"
        },
        {
            "location": [ -6.802374, 62.004142 ],
            "name": "Marknagilsvegur"
        },
        {
            "location": [ 7.419449, 43.731931 ],
            "name": "Avenue du Port"
        }
    ],
    "sources": [
        {
            "location": [ -6.80897, 62.000075 ],
            "name": "Kirkjubøarvegur"
        },
        {
            "location": [ -6.802374, 62.004142 ],
            "name": "Marknagilsvegur"
        },
        {
            "location": [ 7.419449, 43.731931 ],
            "name": "Avenue du Port"
        }
    ]
}
```

### Matrix errors

On error, the server responds with different HTTP status codes. For responses with HTTP status codes lower than `500`, the JSON response body includes the `code` property, which may be used by client programs to manage control flow. The response body may also include a `message` property with a human-readable explanation of the error.

In the case of a server error, the HTTP status code will be `500` or higher and the response will not include a `code` property.

Response body `code` | HTTP status code | Description
--- | --- |---
`Ok` | `200` | Normal success case.
`ProfileNotFound` | `404` | Use a valid profile as described in [Retrieve a matrix](#retrieve-a-matrix).
`InvalidInput` | `422` | The given request was not valid. The `message` key of the response will hold an explanation of the invalid input.

In cases where no route is found between a source and a destination, no error will be returned. Instead, the respective value in the `durations` or `distances` matrix will be `null`.
