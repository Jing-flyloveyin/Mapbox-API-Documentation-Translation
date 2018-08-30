## Map Matching

This documentation is for `v5` of the Map Matching API. For information about the earlier version, see the [`v4` documentation](./pages/map_matching_v4.html).

The Mapbox Map Matching API snaps fuzzy, inaccurate traces from a GPS unit or a phone to the OpenStreetMap road and path network using the Directions API. This produces clean paths that can be displayed on a map or used for other analysis. The Map Matching API can also return a full directions response to queries.

Swift and Objective-C support for Map Matching is provided by the [MapboxDirections.swift](https://github.com/mapbox/MapboxDirections.swift)
library.

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

**Restrictions and limits**

- The Map Matching API is limited to 60 requests per minute.
- Each request can have a maximum of 100 coordinates.
- Results must be displayed on a Mapbox map using one of the Mapbox [libraries or SDKs](https://mapbox.com/developers).

For high volume or other use cases, [contact us](https://mapbox.com/contact/).

### Retrieve a match

```endpoint
GET /matching/v5/{profile}/{coordinates}.json
```

Return a path on the road and path network that is closest to the input traces.

URL parameter | Description
--- | ---
`profile` | A Mapbox Directions routing profile ID. <table><tr><th>Profile ID</th><th>Description</th></tr><tr><td>`mapbox/driving`</td><td>Car travel times, distances, or both.</td></tr><tr><td>`mapbox/walking`</td><td>Pedestrian and hiking travel times, distances, or both</td></tr><tr><td>`mapbox/cycling`</td><td>Bicycle travel times, distances, or both</td></tr><tr><td>`mapbox/driving-traffic`</td><td>Car travel times, distances, or both as informed by traffic data</td></tr></table>
`coordinates` | A semicolon-separated list of `{longitude},{latitude}` coordinate pairs to visit in order. There can be between `2` and `100` coordinates.

You can further refine the results from this endpoint with the following optional parameters:

Query parameter | Description
--- | ---
`annotations`<br /> (optional) | Return additional metadata along the route. Possible values are: `duration`, `distance`, and `speed`. You can include several annotations as a comma-separated list. See the [route leg object](#routeleg-object) for more details on what is included with annotations.
`approaches`<br /> (optional) | A semicolon-separated list indicating the side of the road from which to approach waypoints in a requested route. Accepts `unrestricted` (default, route can arrive at the waypoint from either side of the road) or `curb` (route will arrive at the waypoint on the `driving_side` of the region). If provided, the number of approaches must be the same as the number of waypoints. However, you can skip a coordinate and show its position in the list with the `;` separator. Must be used in combination with `steps=true`.
`geometries`<br /> (optional) | The format of the returned geometry. Allowed values are: `geojson` (as [LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2)), [`polyline`](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) (default, a polyline with precision 5), and [`polyline6`](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) (a polyline with precision 6).
`language`<br /> (optional) | The language of returned turn-by-turn text instructions. See [supported languages](#instructions-languages). The default is `en` (English). Must be used in combination with `steps=true`.
`overview`<br /> (optional) | The type of returned overview geometry. Can be `full` (the most detailed geometry available), `simplified` (default, a simplified version of the full geometry), or `false` (no overview geometry).
`radiuses`<br /> (optional) | The maximum distance a coordinate can be moved to snap to the road network in meters. There must be as many radiuses as there are coordinates in the request, each separated by `;`. Values can be a number between `0.0` and `50.00`. Use higher numbers (`20`-`50`) for noisy traces and lower numbers (`1`-`10`) for clean traces. The default value is `5`.
`steps`<br /> (optional) | Whether to return steps and turn-by-turn instructions (`true`) or not (`false`, default).
`tidy`<br /> (optional) | Whether to remove clusters and re-samples traces for improved map matching results (`true`) or not (`false`, default).
`timestamps`<br /> (optional) | Numbers in [Unix time](https://en.wikipedia.org/wiki/Unix_time) that correspond to each coordinate provided in the request. There must be as many timestamps as there are request coordinates, each separated by `;`, in ascending order. For best results, timestamps should have a sample rate of about 5 seconds.
`waypoint_names`<br /> (optional) | A semicolon-separated list of custom names for coordinates used for the arrival instruction in banners and voice instructions. Values can be any string, and the total number of all characters cannot exceed `500`. The list of `waypoint_names` must be a semicolon-separated list that is the same length as the list of coordinates, but you can skip a coordinate and show its position with the `;` separator.
`waypoints`<br /> (optional) | Indicates which input coordinates should be treated as waypoints. By default, all coordinates result in a waypoint with arrival and departure events in the [match object's](#match-object) route. Can be an index corresponding to any of the input coordinates, but must contain the first (`0`) and last coordinates' index separated by `;`. Most useful in combination with `steps=true` and requests based on traces with high sample rates.

With clean matches, only one match object is returned. When the the algorithm cannot decide the correct match between two points, it will omit that line and return several sub-matches as match objects. The higher the number of sub-match match objects, the more likely it is that the input traces are poorly aligned to the road network.

Some pre-processing tips to achieve the best results:

- Timestamps improve the quality of the matching and are highly recommended.
- The Map Matching API is limited to processing traces with up to 100 coordinates. If you need to process longer traces, you can split the trace and make multiple requests.
- Clusters of points (like a person waiting at a train crossing for a few minutes) often don't add more information to a trace and can negatively impact map-matching quality. We recommend that you tidy the trace (remove clusters and provide a uniform sample rate). You can use the `tidy=true` query parameter or pre-process your traces with external tools like [geojson-tidy](https://github.com/mapbox/geojson-tidy).
- Map matching works best with a sample rate of 5 seconds between points. If your trace has a higher sample rate, you may want to downsample your trace.
- With the `waypoints` parameter specified, traces that would normally return with sub-matches will error. We recommend tidying traces before using them with the `waypoints` parameter.  

#### Example request

```curl
# Basic request that returns a match object with route legs between each waypoint
curl "https://api.mapbox.com/matching/v5/mapbox/driving/-117.17282,32.71204;-117.17288,32.71225;-117.17293,32.71244;-117.17292,32.71256;-117.17298,32.712603;-117.17314,32.71259;-117.17334,32.71254?access_token={your_access_token}"

# Request with the approaches parameter set to 'curb' for each waypoint
curl "https://api.mapbox.com/matching/v5/mapbox/driving/-117.17282,32.71204;-117.17288,32.71225;-117.17293,32.71244;-117.17292,32.71256?approaches=curb;curb;curb;curb&access_token={your_access_token}"


# Request with various parameters, returns a match object with 1 route leg between the first and last waypoints
curl "https://api.mapbox.com/matching/v5/mapbox/driving/2.344003,48.85805;2.34675,48.85727;2.34868,48.85936;2.34955,48.86084;2.34955,48.86088;2.34962,48.86102;2.34982,48.86125?steps=true&tidy=true&waypoints=0;6&waypoint_names=Home;Work&banner_instructions=true&access_token={your_access_token}"
```

```python
service = MapMatcher()
# input data must be a single GeoJSON-like Feature with a LineString geometry
# optional coordTimes property should be an array containing timestamps
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
# response geojson contains a FeatureCollection with a single feature
# with the new LineString corrected to match segments from the selected profile
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
// Not yet supported.
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

#### Example response
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

### The match response object

The **match response object** contains one or more [match objects](#match-object), as well as one or more [tracepoint objects](#tracepoint-object).

Property | Description
--- | ---
`code` | A string indicating the state of the response. The potential values are listed in the [Map Matching status codes section](#map-matching-status-codes).
`matchings` | An array of [match objects](#match-object).
`tracepoints` | An array of [tracepoint objects](#tracepoint-object) that represent the location an input point was matched with, in the order in which they were matched. If a trace point is omitted by the Map Matching API because it is an outlier, the entry will be `null`.

With clean matches, only one match object is returned. When the the algorithm cannot decide the correct match between two points, it will omit that line and return several sub-matches as match objects. The higher the number of sub-match match objects, the more likely it is that the input traces are poorly aligned to the road network.

#### Example match response object

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

### The match object

A **match object** is a [route object](#route-object) with an additional confidence field:

Property | Description
--- | ---
`confidence` | A float indicating the level of confidence in the returned match, from `0` (low) to `1` (high).
`distance` | A float indicating the distance traveled in meters.
`duration` | A float indicating the estimated travel time in seconds.
`weight` | A string indicating the weight in the units described by `weight_name`.
`weight_name` | A string indicating the weight that was used. The default is `routability`, which is duration-based with additional penalties for less desirable maneuvers.
`geometry` | Depending on the `geometries` parameter in the request, this is a [GeoJSON LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2) or a [Polyline string](https://developers.google.com/maps/documentation/utilities/polylinealgorithm). Depending on the `overview` parameter in the request, this is the complete route geometry (`full`), a simplified geometry to the zoom level at which the route can be displayed in full (`simplified`), or is not included (`false`).
`legs` | An array of [route leg objects](#routeleg-object).
`voice_locale` | The locale used for voice instructions. Defaults to `en` (English). See [supported languages](#instructions-languages). Optionally included if `steps=true`.

#### Example match object

```json
{
    "confidence": 0.9548844020537051,
    "distance": 103.7,
    "duration": 16.4,
    "geometry": "gatfEfidjUi@Le@@Y?E??J?^Hf@",
    "legs": []
}
```

### The tracepoint object

A **tracepoint object** is a [waypoint object](#waypoint-object) with 3 additional fields: `matchings_index`, `waypoint_index`, and `alternatives_count`.

Property | Description
--- | ---
`matchings_index` | The index of the match object in `matchings` that the sub-trace was matched to.
`waypoint_index` | The index of the waypoint inside the matched route.
`alternatives_count` | The number of probable alternative matchings for this trace point. A value of `0` indicates that this point was matched unambiguously. Split the trace at these points for incremental map matching.
`name` | The name of the road or path the coordinate snapped to.
`location` | An array that contains the location of the snapped coordinate, in the format `[longitude, latitude]`.

#### Example tracepoint object

```json
{
    "waypoint_index": 0,
    "location": [-117.172836, 32.71204],
    "name": "North Harbor Drive",
    "matchings_index": 0,
    "alternatives_count": 0
}
```

### Map Matching status codes

On error, the server responds with different HTTP status codes.

- For responses with HTTP status codes lower than `500`, the JSON response body includes the code property, which may be used by client programs to manage control flow. The response body may also include a message property, with a human-readable explanation of the error.
- If a server error occurs, the HTTP status code will be `500` or higher and the response will not include a `code` property.

Status code | Description
--- | ---
`Ok` | Normal case
`NoMatch` | The input did not produce any matches, or the `waypoints` requested were not found in the resulting match. `features` will be an empty array.
`TooManyCoordinates` |  There are more than 100 points in the request.
`InvalidInput` | `message` will hold an explanation of the invalid input.
`ProfileNotFound` | Needs to be a valid profile (`mapbox/driving`,  `mapbox/driving-traffic`, `mapbox/walking`, or  `mapbox/cycling`).

### Using HTTP POST

The Map Matching API supports access via the HTTP `POST` method.  To submit a request using HTTP `POST`, you must make the following changes to the request:

  1. The HTTP method must be `POST`
  2. The `Content-Type` of the request must be `application/x-www-form-urlencoded`
  3. The coordinate list must be present in the request body as the `coordinates=` parameter, and there must be no coordinates in the URL
  4. The `access_token` parameter must be part of the `POST` URL, not the body
  5. All other parameters must be part of the request body

An example `POST` request looks like this:

```
POST /matching/v5/mapbox/driving?access_token={your_access_token} HTTP/1.0
Content-Type: application/x-www-form-urlencoded

coordinates=2.344003915786743,48.85805170891599;2.346750497817993,48.85727523615161;2.348681688308716,48.85936462637049;2.349550724029541,48.86084691113991;2.349550724029541,48.8608892614883;2.349625825881958,48.86102337068847;2.34982967376709,48.86125629633996&steps=true&tidy=true&waypoints=0;6
```

```curl
# Basic request using POST interface
$ curl -d "coordinates=-117.17282,32.71204;-117.17288,32.71225;-117.17293,32.71244;-117.17292,32.71256;-117.17298,32.712603;-117.17314,32.71259;-117.17334,32.71254"
"https://api.mapbox.com/matching/v5/mapbox/driving?access_token={your_access_token}"

# Request with various parameters, will return a match object with one route leg between the first and last waypoints
$ curl -d "coordinates=2.344003,48.85805;2.34675,48.85727;2.34868,48.85936;2.34955,48.86084;2.34955,48.86088;2.349625,48.86102;2.34982,48.86125&steps=true&tidy=true&waypoints=0;6&waypoint_names=Home;Work&banner_instructions=true" "https://api.mapbox.com/matching/v5/mapbox/driving?access_token={your_access_token}"
```
