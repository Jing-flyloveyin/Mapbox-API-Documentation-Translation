## Tilequery

<!-- preview -->

The Mapbox Tilequery API allows you to retrieve data about specific features from a vector tileset, based on a given latitude and longitude. The Tilequery API makes it possible to query for features within a radius, do point in polygon queries, query for features in multiple composite layers, and augment data from the [Mapbox Geocoding API](#geocoding) with custom data.

Mapbox Tilequery API has some functional similarities to the reverse geocoding function of the Mapbox Geocoding API, which accepts a `{longitude},{latitude}` pair and returns an address. The Tilequery API returns the location and properties of the features within the query radius.

**Restrictions and limits**

- Use of the Tilequery API endpoint is rate limited based on your user plan. The default is 300 requests per minute.
- Exceeding your user plan's number requests per minute will result in an `HTTP 429 Too Many Requests` response.
- For information on rate limit headers, see the [Rate limits](#rate-limits) section.

If you require a higher rate limit, [contact us](https://www.mapbox.com/contact/).

```javascript
const mbxTilequery = require('@mapbox/mapbox-sdk/services/tilequery');
const tilequeryClient = mbxTilequery({ accessToken: '{your_access_token}' });
```

### The tilequery object

A request to the Tilequery API returns a GeoJSON `FeatureCollection` of features at or near the geographic point described by `{longitude},{latitude}` and within the distance described by the optional `radius` parameter.

Each feature in the response body also includes a `tilequery` object, which contains the following additional properties:

Property | Description
--- | ---
`distance` | The approximate distance in meters from the feature result to the queried point.
`geometry` | The original geometry _type_ of the feature. This can be `”point”`, `”linestring”`, or `”polygon”`. The actual geometry of the feature is not returned in the result set.
`layer` | The vector tile layer of the feature result.

The `geometry.type` in the response will be `Point` for all features returned in the `FeatureCollection`:

-   For `Polygon` and `MultiPolygon` features, the geometry is the point at the queried `{longitude},{latitude}` if the query point exists _within_ the polygon. If the query point is _outside_ the polygon, the geometry is the closest point along the nearest edge of the feature within the `radius` threshold.
-   For `LineString` and `MultiLineString` features, the geometry is the nearest point along the feature within the `radius` threshold of `{longitude},{latitude}`.
-   For `Point` and `MultiPoint` features, the geometry is the nearest point within the `radius` threshold of `{longitude},{latitude}`.

#### The tilequery object

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

### Retrieve features from vector tiles

```endpoint
GET /v4/{map_id}/tilequery/{lon},{lat}.json
```

Use this endpoint to retrieve features from vector tiles.

URL parameter | Description
--- | ---
`map_id` | The ID of the map being queried. To composite multiple layers, provide a comma-separated list of map IDs.
`{lon},{lat}` | The longitude and latitude to be queried.

You can further refine the results from this endpoint with the following optional parameters:

Query parameter | Description
--- | ---
`radius`<br /> (optional) | The approximate distance in meters to query for features. Defaults to `0`. Has no upper bound. Required for queries against point and line data. Due to the nature of tile buffering, a query with a large radius made against equally large point or line data may not include all possible features in the results. Queries will use tiles from the maximum zoom of the tileset, and will only include the intersecting tile plus 8 surrounding tiles when searching for nearby features.
`limit`<br /> (optional) | The number of features between `1`-`50` to return. Defaults to `5`.
`dedupe`<br /> (optional) | Determines whether results will be deduplicated (`true`, default) or not (`false`).                               
`geometry`<br /> (optional) | Queries for a specific geometry type. Options are `polygon`, `linestring`, or `point`.
`layers`<br /> (optional) | A comma-separated list of layers to query, rather than querying all layers. If a specified layer does not exist, it is skipped. If no layers exist, returns an empty `FeatureCollection`.

#### Example requests

```curl
# Retrieve features within a 10 meter radius of the specified location
curl "https://api.mapbox.com/v4/mapbox.mapbox-streets-v7/tilequery/-122.42901,37.80633.json?radius=10&access_token={your_access_token}"

# Return at most 20 features
curl "https://api.mapbox.com/v4/mapbox.mapbox-streets-v7/tilequery/-122.42901,37.80633.json?limit=20&access_token={your_access_token}"

# Query multiple maps
curl "https://api.mapbox.com/v4/{map_id_1},{map_id_2},{map_id_3}/tilequery/-122.42901,37.80633.json&access_token={your_access_token}"

# Return only results from the poi_label and building layers within a 30 meter radius of the specified location
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

#### Example response

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
