## Optimization

The Mapbox Optimization API returns a duration-optimized route between the input coordinates. This is also known as solving the [Traveling Salesperson Problem](https://en.wikipedia.org/wiki/Travelling_salesman_problem). A typical use case for the Optimization API is planning the route for deliveries in a city. You can retrieve a route for car driving, bicycling, and walking.

**Restrictions and limits**

- Maximum 12 coordinates per request
- Maximum 25 distributions per request
- Maximum 60 requests per minute

For higher volumes, [contact us](https://www.mapbox.com/contact/).

### Retrieve an optimization

```endpoint
GET optimized-trips/v1/{profile}/{coordinates}
```

A call to this endpoint retrieves a duration-optimized route between input coordinates.

URL parameter | Description
--- | ---
`profile` | A Mapbox Directions routing profile ID. <table><tr><th>Profile ID</th><th>Description</th></tr><tr><td>`mapbox/driving`</td><td>Car travel times, distances, or both.</td></tr><tr><td>`mapbox/walking`</td><td>Pedestrian and hiking travel times, distances, or both</td></tr><tr><td>`mapbox/cycling`</td><td>Bicycle travel times, distances, or both</td></tr></table> The Optimization API does not support the `mapbox/driving-traffic` profile.
`coordinates` | A semicolon-separated list of `{longitude},{latitude}` coordinates. There must be between 2 and 12 coordinates. The first coordinate is the start and end point of the trip.

You can further refine the results from this endpoint with the following optional parameters:

Query parameter | Description
--- | ---
`annotations`<br /> (optional) | Return additional metadata along the route. You can include several annotations as a comma-separated list. Possible values are: <table><tr><th>`annotations`</th><th>Description</th></tr><tr><td>`duration`</td><td>The duration between each pair of coordinates in seconds</td></tr><tr><td>`distance`</td><td>The distance between each pair of coordinates in meters</td></tr><tr><td>`speed`</td><td>The speed between each pair of coordinates in meters per second</td></tr></table>
`approaches`<br /> (optional) | A semicolon-separated list indicating the side of the road from which to approach waypoints in a requested route. Accepts `unrestricted` (default, route can arrive at the waypoint from either side of the road) or `curb` (route will arrive at the waypoint on the `driving_side` of the region). If provided, the number of approaches must be the same as the number of waypoints. However, you can skip a coordinate and show its position in the list with the `;` separator. Must be used in combination with `steps=true`.
`bearings`<br /> (optional) | Influences the direction in which a route *starts* from a waypoint. Used to filter the road segment on which a waypoint will be placed by direction. This is useful for making sure the new routes of rerouted vehicles continue traveling in their current direction. A request that does this would provide bearing and radius values for the first waypoint and leave the remaining values empty. Must be used in conjunction with the `radiuses` parameter. Takes 2 values per waypoint: an angle clockwise from true north between 0 and 360, and the range of degrees by which the angle can deviate (recommended value is 45° or 90°), formatted as `{angle, degrees}`. If provided, the list of bearings must be the same length as the list of waypoints. However, you can skip a coordinate and show its position in the list with the `;` separator.
`destination`<br />  (optional) | Specify the destination coordinate of the returned route. Accepts `any` (default) or `last`.
`distributions`<br /> (optional) | Specify pick-up and drop-off locations for a trip by providing a `;` delimited list of number pairs that correspond with the `coordinates` list. The first number of a pair indicates the index to the coordinate of the pick-up location in the coordinates list, and the second number indicates the index to the coordinate of the drop-off location in the coordinates list. Each pair must contain exactly 2 numbers, which cannot be the same. The returned solution will visit pick-up locations before visiting drop-off locations. The first location can only be a pick-up location, not a drop-off location.
`geometries`<br /> (optional) | The format of the returned geometry. Allowed values are: `geojson` (as [LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2)), [`polyline`](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) (default, a polyline with precision 5), [`polyline6`](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) (a polyline with precision 6).
`language`<br /> (optional) | The language of returned turn-by-turn text instructions. See [supported languages](#instructions-languages). The default is `en` (English).
`overview`<br /> (optional) | The type of the returned overview geometry. Can be `full` (the most detailed geometry available), `simplified` (default, a simplified version of the full geometry), or `false` (no overview geometry).
`radiuses`<br /> (optional) | The maximum distance a coordinate can be moved to snap to the road network in meters. There must be as many radiuses as there are coordinates in the request, each separated by `;`. Values can be any number greater than `0` or the string `unlimited`. A `NoSegment` error is returned if no routable road is found within the radius.
`source`<br />  (optional) | The coordinate at which to start the returned route. Accepts `any` (default) or `first`.
`steps`<br /> (optional) | Whether to return steps and turn-by-turn instructions (`true`) or not (`false`, default).
`roundtrip`<br />  (optional) | Indicates whether the returned route is roundtrip, meaning the route returns to the first location (`true`, default) or not (`false`). If `roundtrip=false`, the `source` and `destination` parameters are required but not all combinations will be possible. See the *Fixing Start and End Points* section below for additional notes.

Unrecognized options in the query string result in an `InvalidInput` error.

Note that routes returned by the Optimization API will behave as if [`continue_straight=false`](#retrieve-directions) was set at each waypoint, meaning that the route will continue in the same direction of travel. See the [`continue_straight`](#retrieve-directions) parameter in the Directions API for more details on what this means for the route.

**Fixing Start and End Points**

It is possible to explicitly set the start or end coordinate of the trip:

- When `source=first`, the first coordinate is used as the start coordinate of the trip in the output.
- When `destination=last`, the last coordinate is used as the destination coordinate of the trip in the output.
- If you specify `any` for `source` or `destination`, any of the coordinates can be used as the first or last coordinate in the output.
- If `source=any&destination=any`, the returned roundtrip will start at the first input coordinate by default.

Not all combinations of `roundtrip`, `source`, and `destination` are supported. Right now, the following combinations are possible:

| roundtrip | source | destination | supported |
| :-- | :-- | :-- | :-- |
| true | first | last | **yes** |
| true | first | any | **yes** |
| true | any | last | **yes** |
| true | any | any | **yes** |
| false | first | last | **yes** |
| false | first | any | no |
| false | any | last | no |
| false | any | any | no |

#### Example requests

```curl
# Request an optimized car trip with no additional options
curl "https://api.mapbox.com/optimized-trips/v1/mapbox/driving/-122.42,37.78;-122.45,37.91;-122.48,37.73?access_token={your_access_token}"

# Request an optimized bicycle trip with steps and a GeoJSON response
curl "https://api.mapbox.com/optimized-trips/v1/mapbox/cycling/-122.42,37.78;-122.45,37.91;-122.48,37.73?steps=true&geometries=geojson&access_token={your_access_token}"

# Request an optimized car roundtrip in Berlin with four coordinates, starting at the first coordinate pair and ending at the last coordinate pair
curl "https://api.mapbox.com/optimized-trips/v1/mapbox/driving/13.388860,52.517037;13.397634,52.529407;13.428555,52.523219;13.418555,52.523215?source=first&destination=last&roundtrip=true&access_token={your_access_token}"

# Request an optimized car trip with four coordinates and one distributions constraint where the last given coordinate must be visited before the second
curl "https://api.mapbox.com/optimized-trips/v1/mapbox/driving/13.388860,52.517037;13.397634,52.529407;13.428555,52.523219;13.418555,52.523215?roundtrip=true&distributions=3,1&access_token={your_access_token}"

# Request an optimized car trip with specified waypoint approach bearings and turn-by-turn instructions
curl "https://api.mapbox.com/optimized-trips/v1/mapbox/driving/-122.42,37.78;-122.45,37.91;-122.48,37.73?radiuses=unlimited;unlimited;unlimited&bearings=45,90;90,1;340,45&steps=true&access_token={your_access_token}"
```

```python
# This API is not available through the Python SDK
```

```javascript
// This API is not available through the JavaScript SDK
```

```bash
# This API is not available through the Mapbox CLI
```

```java
MapboxOptimization optimizedClient = MapboxOptimization.builder()
  .source(DirectionsCriteria.SOURCE_FIRST)
  .destination(DirectionsCriteria.DESTINATION_ANY)
  .coordinates(listOfPoints)
  .overview(DirectionsCriteria.OVERVIEW_FULL)
  .profile(DirectionsCriteria.PROFILE_DRIVING)
  .accessToken("{your_access_token}")
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
    "waypoints": [
        {
            "name": "North Lake Boulevard",
            "location": [ -120.141159, 39.170872 ],
            "waypoint_index": 0,
            "trips_index": 0
        },
        {
            "name": "Virginia Drive",
            "location": [ -120.14984, 39.159985 ],
            "waypoint_index": 2,
            "trips_index": 0
        },
        {
            "name": "Fairway Drive",
            "location": [ -120.150648, 39.340689 ],
            "waypoint_index": 1,
            "trips_index": 0
        }
    ],
    "trips": [
        {
            "geometry": "}panFfah|Ujj@ru@`Dp`BwNdpAwc@pw@ibAxm@snA|Ic^|\\q{@{S}`@lVewBzUa}@t^}oAdEkbB}[{{AvHqdDs\\qn@lV_OqpA{`A}bBaRucA_gB}eCbI_MzXzY`Va]xUbPzE|[{E}[_WoP{Tl]{X{YoLfQtjBvdCbRpcA|_A|`BbD`qA~y@{WpdDr\\z{AwHh}A`\\hsA{Dv~@c_@dwB{U|`@mVp{@zSfe@_`@niA}G|aAin@~b@mx@|LunBsBw}@{Sia@bp@nDcCpYbCqYou@uDsP_U",
            "legs": [
                {
                    "summary": "",
                    "weight": 1962.8,
                    "duration": 1876.9,
                    "steps": [],
                    "distance": 31507.9
                },
                {
                    "summary": "",
                    "weight": 2211.9,
                    "duration": 2035.1,
                    "steps": [],
                    "distance": 32720.5
                },
                {
                    "summary": "",
                    "weight": 283.5,
                    "duration": 238.1,
                    "steps": [],
                    "distance": 1885.2
                }
            ],
            "weight_name": "routability",
            "weight": 4458.2,
            "duration": 4150.1,
            "distance": 66113.6
        }
    ]
}
```

### Optimization response object
The response to an Optimization API request is a JSON object that contains the following properties:

Property | Description
--- | ---
`code` | A string indicating the state of the response. This is a separate code than the HTTP status code. On normal valid responses, the value will be `Ok`.
`waypoints` | An array of `waypoint` objects. Each waypoint is an input coordinate snapped to the road and path network. The waypoints appear in the array in the order of the input coordinates.
`trips` | An array of 0 or 1 `trip` objects.

**Waypoint objects**

A **waypoint object** is an input coordinate snapped to the roads network that contains the following properties:

Property | Description
--- | ---
`name` | A string with the name of the road or path that the input coordinate snapped to.
`location` | An array containing the `[longitude, latitude]` of the snapped coordinate.
`trips_index` | The index of the `trip` object that contains this waypoint in the `trips` array.
`waypoint_index` | The index of the position of the given waypoint within the `trip`.

**Trip object**

A **trip object** describes a route through multiple waypoints, and has the following properties:

Property | Description
--- | ---
`geometry` | Depending on the `geometries` parameter, this is either a [GeoJSON LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2) or a [Polyline string](https://developers.google.com/maps/documentation/utilities/polylinealgorithm). Depending on the `overview` parameter, this is the complete route geometry (`full`), a simplfied geometry to the zoom level at which the route can be displayed in full (`simplified`), or is not included (`false`).
`legs` | An array of [route leg](#routeleg-object) objects.
`weight_name` | A string indicating which weight was used. The default is `routability`, which is duration-based, with additional penalties for less desirable maneuvers.
`weight` | A float indicating the weight in the units described by `weight_name`.
`duration` | A float indicating the estimated travel time in seconds.
`distance` | A float indicating the distance traveled in meters.

A trip object has the same format as a [route object](#route-object) in the Directions API.

#### Example response object

```json
{
    "code": "Ok",
    "waypoints": [
        {
            "location": [ -6.80897, 62.000075 ],
            "waypoint_index": 0,
            "name": "Kirkjubøarvegur",
            "trips_index": 0
        },
        {
            "location": [ -6.802374, 62.004142 ],
            "waypoint_index": 1,
            "name": "Marknagilsvegur",
            "trips_index": 0
        }
    ],
    "trips": [
        {
            "distance": 1660.8,
            "duration": 153,
            "legs": [
                {
                    "summary": "",
                    "duration": 77.3,
                    "steps": [],
                    "distance": 830.4
                },
                {
                    "distance": 830.4,
                    "steps": [],
                    "duration": 75.7,
                    "summary": ""
                }
            ],
            "geometry": "oklyJ`{ph@yBuY_F{^_FxJoBrBs@d@mATlAUr@e@nBsB~EyJ~Ez^xBtY"
        }
    ]
}
```

### Optimization errors

On error, the server responds with different HTTP status codes. For responses with HTTP status codes lower than `500`, the JSON response body includes the `code` property, which may be used by client programs to manage control flow. The response body may also include a `message` property with a human-readable explanation of the error.

If a server error occurs, the HTTP status code will be `500` or higher and the response will not include a `code` property.

Response body `code` | HTTP status code | Description
--- | --- |---
`Ok` | `200` | Normal success case.
`NoTrips` | `200` | For one coordinate, no route to other coordinates could be found. Check for impossible routes (for example, routes over oceans without ferry connections).
`NotImplemented`  | `200` | For the given combination of `source`, `destination`, and `roundtrip`, this request is not supported.
`ProfileNotFound` | `404` | Use a valid profile as described in [Retrieve an optimization](#retrieve-an-optimization).
`InvalidInput` | `422` | The given request was not valid. The `message` key of the response will hold an explanation of the invalid input.

All other properties might be undefined.
