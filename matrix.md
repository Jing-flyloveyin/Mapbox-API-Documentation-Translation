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
`approaches`<br /> (可选) | 一个分号分隔的列表，表示请求的路线中最接近航路点的道路。接受`unrestricted`(默认情况下，路线可以从道路两侧到达航路点)或`curb`(路线将到达该区域的`driving_side`的航路点)。如果提供的话，方法的数量必须与航路点的数量相同。但是，您可以略过坐标并用`;` 分隔符显示列表中的位置。
`destinations`<br /> (可选) | 使用给定索引的坐标作为目的地。可能的值是：分号分隔基于0的索引列表，或`all`（默认值）。选项`all`允许使用所有坐标作为目的地。
`sources`<br /> (可选) | 使用给定索引的坐标作为源地。可能的值是：分号分隔基于0的索引列表，或`all`（默认值）。选项`all`允许使用所有坐标作为源地。

查询字符串中的未识别选项将导致`InvalidInput`错误。

#### 请求示例

```curl
# 请求一个对称的3x3矩阵，用于最靠近路边的车辆。
curl "https://api.mapbox.com/directions-matrix/v1/mapbox/driving/-122.42,37.78;-122.45,37.91;-122.48,37.73?approaches=curb;curb;curb&access_token={your_access_token}"

# 请求一个不对称2x3矩阵用于自行车。
curl "https://api.mapbox.com/directions-matrix/v1/mapbox/cycling/-122.42,37.78;-122.45,37.91;-122.48,37.73?sources=0;2&destinations=all&access_token={your_access_token}"

# 请求一个包括持续时间和距离的1x3矩阵，用于步行。
curl "https://api.mapbox.com/directions-matrix/v1/mapbox/walking/-122.418563,37.751659;-122.422969,37.75529;-122.426904,37.759617?sources=1&annotations=distance,duration&access_token={your_access_token}"
```

```python
service = DirectionsMatrix()
# 输入的路径点是一些要素，通常像GeoJSON一样。
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
# 矩阵方法可以用点要素和移动配置的profile来调用。
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
# 此API无法通过Mapbox CLI获得。
```

```java
MapboxMatrix directionsMatrixClient = MapboxMatrix.builder()
  .accessToken("{your_access_token}")
  .profile(DirectionsCriteria.PROFILE_DRIVING)
  .coordinates(listOfPoints)
  .build();
```

```objc
// Mapbox Objective-C libraries无法访问此API
```

```swift
// Mapbox Swift libraries无法访问此API
```

#### 响应结果示例
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

### 矩阵响应对象

一个Matrix API请求的响应的结果包含下列属性的JSON对象：

属性 | 描述
--- | ---
`code` | 响应状态代码。这是一个区别于HTTP状态代码的代码。在正常时，该值将为`Ok`。有关更多信息，请参阅下面的错误部分。
`durations` | 持续时间作为数组中的一个数组，以行-主序表示矩阵。`durations[i][j]`给出了从i<sup>th</sup>源地到j<sup>th</sup>目的地的移动时间。所有的值单位是秒。同一坐标之间的持续时间总是`0`。如果无法找到持续时间，则结果为`null`。
`distances` | 距离作为数组中的一个数组，以行-主序表示矩阵。`distances[i][j]`给出了从i<sup>th</sup>源地到j<sup>th</sup>目的地的距离。所有值的单位是米。同一坐标之间的距离总是`0`。如果无法找到距离，则结果为`null`。
`destinations:` | `waypoint`对象中的一个数组。每个航路点是一个输入到道路和路径网络的坐标。航路点按照输入坐标的顺序出现在数组中，或在`destinations`查询参数指定的顺序出现在数组中。
`sources` | `waypoint`对象中的一个数组。每个航路点是一个输入到道路和路径网络的坐标。航路点按照输入坐标的顺序出现在数组中，或在`sources`查询参数指定的顺序出现在数组中。

**注意:** 如果在源地和目的地之间没有找到路线，则在`durations`或`distances`矩阵中的相应值将是`null`。

#### 矩阵响应对象示例

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

### 矩阵错误

在错误时，服务器响应不同的HTTP状态代码。对于HTTP状态码低于`500`的响应，JSON响应主体包括`code`属性，客户端程序可以使用该属性来管理控制流。响应体还可以包括`message`属性，并对错误进行人类可读的解释。

在服务器错误的情况下，HTTP状态代码将是`500`或更高，并且响应将不包括`code`属性。

响应体 `code` | HTTP 状态代码 | 描述
--- | --- |---
`Ok` | `200` | 表示正常，成功返回。
`ProfileNotFound` | `404` | 使用一个有效的配置profile如 [Retrieve a matrix](#retrieve-a-matrix)。
`InvalidInput` | `422` | 给定的请求无效。响应的`message`键将对无效输入进行解释。 

在源地和目的地之间没有找到路径的情况下，将不会返回任何错误。反而，`durations` 或 `distances`在矩阵中的相应值将是`null`。
