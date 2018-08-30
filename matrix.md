## Matrix

The Matrix API returns travel times between many points.

For example, given 3 locations A, B, and C, the Matrix API will return a matrix of all travel times in seconds between the locations:

| |   A   |   B   |   C
--| ----- | ----- | -----
A | A → A | A → B | A → C
B | B → A | B → B | B → C
C | C → A | C → B | C → C

The Matrix API will always return the duration or the distance on the fastest route for each element in the matrix, where an element is an origin-destination pair in the matrix. The Matrix API returns durations in seconds and distances in meters. It does not return route geometries.

Durations or distances between points may not be symmetric, since the routes may differ by direction due to one-way streets or turn restrictions. For example, A to B may have a different duration than B to A.

The Matrix API allows you to efficiently check the reachability of coordinates from each other, filter points by travel time, or run your own algorithms for solving optimization problems.

**Restrictions and limits**
- For the `mapbox/driving`, `mapbox/walking`, and `mapbox/cycling` profiles:
    - Maximum 25 input coordinates per request
    - Maximum 60 requests per minute
- For the `mapbox/driving-traffic` profile:   
    - Maximum 10 input coordinates per request
    - Maximum 30 requests per minute

For higher volumes, [contact us](https://www.mapbox.com/contact/).

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

Returns a duration matrix, a distance matrix, or both, showing travel times and distances between coordinates. In the default case, this endpoint returns a symmetric matrix that uses all the input coordinates as sources and destinations. Using the optional `sources` and `destination` parameters, you can also generate an asymmetric matrix that uses only some coordinates as sources or destinations.

URL parameter | Description
--- | ---
`profile` | A Mapbox Directions routing profile ID. <table><tr><th>Profile ID</th><th>Description</th></tr><tr><td>`mapbox/driving`</td><td>Car travel times, distances, or both.</td></tr><tr><td>`mapbox/walking`</td><td>Pedestrian and hiking travel times, distances, or both</td></tr><tr><td>`mapbox/cycling`</td><td>Bicycle travel times, distances, or both</td></tr><tr><td>`mapbox/driving-traffic`</td><td>Car travel times, distances, or both as informed by traffic data</td></tr></table>
`coordinates` | A semicolon-separated list of `{longitude},{latitude}` coordinates. There must be between 2 and 25 coordinates.

You can further refine the results from this endpoint with the following optional parameters:

Query parameter | Description
--- | ---
`annotations`<br /> (optional) | Used to specify the resulting matrices. Possible values are: `duration` (default), `distance`, or both values separated by comma.
`approaches`<br /> (optional) | A semicolon-separated list indicating the side of the road from which to approach waypoints in a requested route. Accepts `unrestricted` (default, route can arrive at the waypoint from either side of the road) or `curb` (route will arrive at the waypoint on the `driving_side` of the region). If provided, the number of approaches must be the same as the number of waypoints. However, you can skip a coordinate and show its position in the list with the `;` separator.
`destinations`<br /> (optional) | Use the coordinates at a given index as destinations. Possible values are: a semicolon-separated list of 0-based indices, or `all` (default). The option `all` allows using all coordinates as destinations.
`sources`<br /> (optional) | Use the coordinates at a given index as sources. Possible values are: a semicolon-separated list of 0-based indices, or `all` (default). The option `all` allows using all coordinates as sources.

Unrecognized options in the query string will result in an `InvalidInput` error.

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
