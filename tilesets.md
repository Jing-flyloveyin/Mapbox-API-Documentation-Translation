## Tilesets 瓦片集

Mapbox Tilesets API可以用来读取栅格和矢量[瓦片集](https://www.mapbox.com/help/define-tileset/)的元数据。如果想要请求瓦片，请使用[Maps API](#maps)。

**约束与限制**

- Tilesets API每分钟最多请求50次。

- 必须使用HTTPS来请求，不支持HTTP。

```javascript
const mbxTilesets = require('@mapbox/mapbox-sdk/services/tilesets');
const tilesetsClient = mbxTilesets({ accessToken: '{your_access_token}' });
```
### 瓦片集对象

请求Tilesets API将返回一个或多个瓦片集对象，每个瓦片集对象都包含以下属性：

属性 | 描述
--- | ---
`type` | 所含数据的类型，`raster` 或 `vector`。
`center` | 所含数据中心所在的经纬度和缩放级别，格式为 `[lon, lat, zoom]`。
`created` | 表示瓦片集创建时间的时间戳。
`description` | 瓦片集的描述信息。
`filesize` | 瓦片集所占用的字节数。
`id` | 瓦片集的唯一标识。
`modified` | 表示瓦片集最近一次修改时间的时间戳。
`name` | 瓦片集的名称。
`visibility` | 瓦片集的访问权限，`public` 或 `private`。
`status` | 瓦片集的处理状态，值包含： `available`、 `pending` 和 `invalid`。


#### 瓦片集对象示例

```json
{
  "type": "vector",
  "center": [-0.2680000000000007, 11.7014165, 2],
  "created": "2015-09-09T23:30:17.936Z",
  "description": "",
  "filesize": 17879790,
  "id": "mapbox.05bv6e12",
  "modified": "2015-09-09T23:30:17.906Z",
  "name": "routes.geojson",
  "visibility": "public",
  "status": "available"
}
```

### 瓦片集列表

```endpoint
GET /tilesets/v1/{username} tilesets:list
```

列出归属于指定账户的所有瓦片集。此端点支持[分页](#pagination)。

URL参数 | 描述
--- | ---
`username` | 要列出的瓦片集所属账户的用户名

你可以使用以下可选参数进一步细化这个端点的结果：

查询参数 | 描述
----------|------------
`type` | 根据瓦片集类型过滤结果，`raster` 或者 `vector`。
`visibility` | 根据访问权限过滤结果，`public` 或者 `private`。私有瓦片集需要一个属于所有者的访问令牌，公共瓦片集则可以被任意用户的访问令牌请求。 
`sortby` | 根据`created` 或 `modified` 时间戳来排序。
`limit` | 限制返回瓦片集的最大数量，从 `1` 到 `500`。默认为 `100`。
`start` | 设置从哪个瓦片集开始罗列，key可以在响应的`Link`头部中找到。 详见[分页](#pagination)部分。

#### 请求示例

```curl
curl "https://api.mapbox.com/tilesets/v1/{username}?access_token={your_access_token}"

# 请求数量上限为25个且最新创建的矢量瓦片集
curl "https://api.mapbox.com/tilesets/v1/{username}?type=vector&limit=25&sortby=created&access_token={your_access_token}"

# 只请求key为abc123及其之后的瓦片集
curl "https://api.mapbox.com/tilesets/v1/{username}?start=abc123&access_token={your_access_token}"
```

```bash
# 该APi不支持Mapbox CLI
```

```javascript
tilesetsClient
  .listTilesets()
  .then(response => {
    const tilesets = response.body;
  });

tilesetsClient
  .listTilesets()
  .eachPage((error, response, next) => {
    // 处理错误、响应以及调用next。
  });
```

```python
# 该API不支持Mapbox Python SDK
```

```java
// 该API不支持Mapbox Java SDK
```

```objc
// 该API不支持Mapbox Objective-C库
```

```swift
// 该API不支持Mapbox Swift库
```

#### 响应示例

```json
[
  {
    "type": "vector",
    "center": [-0.2680000000000007, 11.7014165, 2],
    "created": "2015-09-09T23:30:17.936Z",
    "description": "",
    "filesize": 17879790,
    "id": "mapbox.05bv6e12",
    "modified": "2015-09-09T23:30:17.906Z",
    "name": "routes.geojson",
    "visibility": "public",
    "status": "available"
  },
  {
    "type": "raster",
    "center": [-110.32479628173822, 44.56501277250615, 8],
    "created": "2016-12-10T01:29:37.682Z",
    "description": "",
    "filesize": 794079,
    "id": "mapbox.4umcnx2j",
    "modified": "2016-12-10T01:29:37.289Z",
    "name": "sample-4czm7e",
    "visibility": "private",
    "status": "available"
  }
]
```
