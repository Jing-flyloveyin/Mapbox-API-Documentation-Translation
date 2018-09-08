## 切片查询

<!-- preview -->

Mapbox的Tilequery API允许您根据给定的经纬度从矢量切片中检索特定的要素数据集。Tilequery API可以查询半径内的要素，判断点是否在多边形内，查询多个复合图层中的要素，使用自定义的数据扩充 [Mapbox Geocoding API](#geocoding)返回的数据。

Mapbox的Tilequery API与Geocoding API中接受一对`{经度}，{纬度}`返回一个地址的反地理编码功能有相似之处。Tilequery API返回查询半径内的各要素的位置和属性。

**约束与限制**

- 根据您不同的用户计划，Tilequery API在终端的使用权限有所不同。默认为每分钟300个请求。
- 每分钟请求量超过用户计划的，将得到`HTTP 429 Too Many Requests`的响应结果。
- 关于权限的更多信息，请参阅 [Rate limits](#rate-limits) 相关章节。

如果您需要更高的权限，[请联系我们](https://www.mapbox.com/contact/)。

```javascript
const mbxTilequery = require('@mapbox/mapbox-sdk/services/tilequery');
const tilequeryClient = mbxTilequery({ accessToken: '{your_access_token}' });
```

### 切片查询结果对象

对Tilequery API的请求返回一个GeoJSON格式的`要素集合`，该集合的各要素为处于以`{经度}，{纬度}`描述的地理坐标点或是在可选的`半径`参数描述距离内的查询结果。

响应体中的每个要素还包含一个`tilequery`对象，该对象包含以下附加属性：

属性 | 说明
--- | ---
`distance` | 要素结果到查询点的大概距离（以米为单位）。
`geometry` | 要素的原始几何 _类型_ 。它可能是`point`，`linestring`，或者`polygon`。结果集中不返回要素的实际几何图形数据。
`layer` | 结果要素的矢量切片图层。

对于返回的`要素集`中所有的要素，其`geometry`的`type`属性值都是`point`：

-   对于`Polygon`和`MultiPolygon`类型的要素而言，如果查询的点在多边形内（_within_），则该点就是查询点`{经度}，{纬度}`。如果查询的点在多边形外（ _outside_ ），则该点是在“半径”阈值内距离该多边形要素最近边的最近点。
-   对于`LineString`和`MultiLineString`类型的要素而言，该点是在`半径`阈值内沿着线要素距离查询点`{经度}，{纬度}`最近的点。
-   对于`Point`和`MultiPoint`类型的要素而言， 该点是在`半径`阈值内距离查询点`{经度}，{纬度}`最近的点。

#### 切片查询结果对象

```json
{
    "type": "FeatureCollection",
    "features": [
        {
            "type": "Feature",
            "geometry": {
                "type": "Point",
                "coordinates": [
                    -122.42901,
                    37.80633
                ]
            },
            "properties": {
                "class": "wood",
                "type": "wood",
                "tilequery": {
                    "distance": 0,
                    "geometry": "polygon",
                    "layer": "landuse"
                }
            }
        },
        {
            "type": "Feature",
            "geometry": {
                "type": "Point",
                "coordinates": [
                    -122.42901,
                    37.80633
                ]
            },
            "properties": {
                "class": "national_park",
                "type": "national_park",
                "tilequery": {
                    "distance": 0,
                    "geometry": "polygon",
                    "layer": "landuse_overlay"
                }
            }
        }
    ]
}

```

### 从矢量切片取回要素集

```endpoint
GET /v4/{map_id}/tilequery/{lon},{lat}.json
```

使用此终端地址从矢量切片检索要素。

URL 参数 | 说明
--- | ---
`map_id` | 要查询的地图ID。对于多个图层而言，请提供以逗号分隔的地图ID列表。
`{lon},{lat}` | 要查询的经纬度。

您可以使用以下可选参数进一步优化查询结果：

查询参数 | 说明
--- | ---
`radius`<br /> (可选) | 查询要素的近似距离（以米为单位）。默认值为`0`，没有上限。对于查询点和线数据而言为必填参数。由于切片缓冲的性质，以很大半径进行查询时会包含同样大量的点和线数据，但返回结果集中无法包含所有可能的要素。查询将使用切片集合中最大层级的切片数据，并且在搜索附近的要素时，仅查询相交的切片块和8个与之相邻的切片块。
`limit`<br /> (可选) | 返回要素的数量（`1`-`50`之间）。默认值为`5`。
`dedupe`<br /> (可选) | 确定结果是否删除重复的数据，默认值为`true`（删除），`false`则不删除。                           
`geometry`<br /> (可选) | 查询指定的几何类型。可选择`polygon`，`linestring`，或者`point`。
`layers`<br /> (可选) | 要查询的图层列表（以逗号分隔开），而不是查询所有图层。如果指定的图层不存在，则跳过该图层。如果图层都不存在，则返回空的`要素集`。

#### 请求示例

```curl
# 获取指定位置10m半径范围内的要素
curl "https://api.mapbox.com/v4/mapbox.mapbox-streets-v7/tilequery/-122.42901,37.80633.json?radius=10&access_token={your_access_token}"

# 返回最多20条要素
curl "https://api.mapbox.com/v4/mapbox.mapbox-streets-v7/tilequery/-122.42901,37.80633.json?limit=20&access_token={your_access_token}"

# 查询多个地图
curl "https://api.mapbox.com/v4/{map_id_1},{map_id_2},{map_id_3}/tilequery/-122.42901,37.80633.json&access_token={your_access_token}"

# 仅返回poi_label和building图层中，指定位置30m半径范围内的要素
curl "https://api.mapbox.com/v4/mapbox.mapbox-streets-v7/tilequery/-122.42901,37.80633.json?radius=30&layers=poi_label,building&access_token={your_access_token}"
```

```javascript
tilequeryClient
  .listFeatures({
    mapIds: ['mapbox.mapbox-streets-v7'],
    coordinates: [-122.42901, 37.80633],
    radius: 10
  })
  .send()
  .then(response => {
    const features = response.body;
  });
```

```python
# This API cannot be accessed with the Python SDK
```

```bash
# This API cannot be accessed with Mapbox CLI
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

#### 查询结果示例

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [
          -122.42901,
          37.80633
        ]
      },
      "properties": {
        "class": "national_park",
        "type": "national_park",
        "tilequery": {
          "distance": 0,
          "geometry": "polygon",
          "layer": "landuse_overlay"
        }
      }
    },
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [
          -122.4289919435978,
          37.806283149631255
        ]
      },
      "properties": {
        "localrank": 1,
        "maki": "park",
        "name": "Fort Mason",
        "name_ar": "Fort Mason",
        "name_de": "Fort Mason",
        "name_en": "Fort Mason Center",
        "name_es": "Fort Mason",
        "name_fr": "Fort Mason",
        "name_pt": "Fort Mason",
        "name_ru": "Fort Mason",
        "name_zh": "梅森堡",
        "name_zh-Hans": "梅森堡",
        "ref": "",
        "scalerank": 1,
        "type": "National Park",
        "tilequery": {
          "distance": 5.4376397785894595,
          "geometry": "point",
          "layer": "poi_label"
        }
      }
    }
  ]
}
```
