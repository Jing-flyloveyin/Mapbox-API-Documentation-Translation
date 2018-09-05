## Tilesets

Mapbox Tilesets API可以用来读取栅格和矢量[tilesets](https://www.mapbox.com/help/define-tileset/)的元数据。如果想要请求瓦片，请使用[Maps API](#maps)。

**约束与限制**

- Tilesets API每分钟最多请求50次。

- 必须使用HTTPS来请求，不支持HTTP。

```javascript
const mbxTilesets = require('@mapbox/mapbox-sdk/services/tilesets');
const tilesetsClient = mbxTilesets({ accessToken: '{your_access_token}' });
```
### 瓦片集对象

请求Tilesets API返回一个或多个瓦片集对象，每个瓦片集对象都包含以下属性：

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
`username` | 要列出瓦片集的账户的用户名

你可以使用以下可选参数进一步细化这个端点的结果：

查询参数 | 描述
----------|------------
`type` | Filter results by tileset type, either `raster` or `vector`.
`visibility` | Filter results by visibility, either `public` or `private`. Private tilesets require an access token that belong to the owner. Public tilesets can be requested with any user's access token.
`sortby` | Sort the listings by their `created` or `modified` timestamps.
`limit` | The maximum number of tilesets to return, from `1` to `500`. The default is `100`.
`start` | The tileset after which to start the listing. The key is found in the `Link` header of a response. See the [pagination](#pagination) section for details.

#### 请求示例

```curl
curl "https://api.mapbox.com/tilesets/v1/{username}?access_token={your_access_token}"

# Limit the results to the 25 most recently created vector tilesets
curl "https://api.mapbox.com/tilesets/v1/{username}?type=vector&limit=25&sortby=created&access_token={your_access_token}"

# Limit the results to the tilesets after the tileset with a start key of abc123
curl "https://api.mapbox.com/tilesets/v1/{username}?start=abc123&access_token={your_access_token}"
```

```bash
# This API cannot be accessed with the Mapbox CLI
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
    // Handle error or response and call next.
  });
```

```python
# This API cannot be accessed with the Mapbox Python SDK
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
