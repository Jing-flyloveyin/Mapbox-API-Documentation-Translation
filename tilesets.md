## Tilesets

The Mapbox Tilesets API supports reading metadata for raster and vector [tilesets](https://www.mapbox.com/help/define-tileset/). To request tiles, use the [Maps API](#maps) instead.

**Restrictions and limits**

- The Tilesets API is limited to 50 requests per minute.
- Requests must be made over HTTPS. HTTP is not supported.

```javascript
const mbxTilesets = require('@mapbox/mapbox-sdk/services/tilesets');
const tilesetsClient = mbxTilesets({ accessToken: '{your_access_token}' });
```

### The tileset object

A request to the Tilesets API returns one or more tileset objects. Each tileset object contains the following properties:

Property | Description
--- | ---
`type` | The kind of data contained, either `raster` or `vector`.
`center` | The longitude, latitude, and zoom level for the center of the contained data, given in the format `[lon, lat, zoom]`.
`created` | A timestamp indicating when the tileset was created.
`description` | A human-readable description of the tileset.
`filesize` | The storage in bytes consumed by the tileset.
`id` | The unique identifier for the tileset.
`modified` | A timestamp indicating when the tileset was last modified.
`name` | The name of the tileset.
`visibility` | The access control for the tileset, either `public` or `private`.
`status` | The processing status of the tileset, one of: `available`, `pending`, or `invalid`.

#### The tileset object

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

### List tilesets

```endpoint
GET /tilesets/v1/{username} tilesets:list
```

List all the tilesets that belong to a specific account. This endpoint supports [pagination](#pagination).

URL parameters | Description
--- | ---
`username` | The username of the account for which to list tilesets

You can further refine the results from this endpoint with the following optional parameters:

Query Parameter | Description
----------|------------
`type` | Filter results by tileset type, either `raster` or `vector`.
`visibility` | Filter results by visibility, either `public` or `private`. Private tilesets require an access token that belong to the owner. Public tilesets can be requested with any user's access token.
`sortby` | Sort the listings by their `created` or `modified` timestamps.
`limit` | The maximum number of tilesets to return, from `1` to `500`. The default is `100`.
`start` | The tileset after which to start the listing. The key is found in the `Link` header of a response. See the [pagination](#pagination) section for details.

#### Example request

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

#### Example response

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
