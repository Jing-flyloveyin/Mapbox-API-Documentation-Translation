## Directions

This documentation is for `v5` of the Directions API. For information about the earlier version, see the [`v4` documentation](./pages/directions-v4.html).

The Mapbox Directions API will show you how to get where you're going. With the Directions API, you can:

- Calculate optimal driving, walking, and cycling routes using traffic- and incident-aware routing
- Produce turn-by-turn instructions
- Produce routes with up to 25 coordinates anywhere on earth

This API supports 4 different routing profiles:

Profile | Description
--- | ---
`mapbox/driving-traffic` | For automotive routing. This profile factors in current and historic traffic conditions to avoid slowdowns. Traffic information is available in [these supported geographies](./pages/traffic-countries.html).
`mapbox/driving` | For automotive routing. This profile shows the fastest routes by preferring high-speed roads like highways.
`mapbox/walking` | For pedestrian and hiking routing. This profile shows the shortest path by using sidewalks and trails.
`mapbox/cycling` | For bicycle routing. This profile shows routes that are short and safer for cyclists by avoiding highways and preferring streets with bike lanes.

Swift and Objective-C support for Directions is provided by the [MapboxDirections.swift](https://github.com/mapbox/MapboxDirections.swift)
library.

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

```python
from mapbox import Directions
```

```javascript
const mbxDirections = require('@mapbox/mapbox-sdk/services/directions');
const directionsClient = mbxDirections({ accessToken: '{your_access_token}' });
```

**Restrictions and limits**

- Requests using the `driving`, `walking`, and `cycling` profiles can specify up to 25 total waypoints (input coordinates that are snapped to the roads network) along the route.
- Requests using the `driving-traffic` profile can specify up to 3 waypoints along the route.
- Traffic coverage for the `driving-traffic` profile is available in [supported geographies](./pages/traffic-countries.html). Requests to this profile revert to `driving` profile results for areas without traffic coverage.
- Maximum 60 requests per minute.
- Maximum total of 10,000 kilometers between all waypoints.

### Retrieve directions

```endpoint
GET /directions/v5/{profile}/{coordinates}
```

Retrieve directions between waypoints. Directions requests must specify at least two waypoints as starting and ending points.

Try this in the [API Playground](https://www.mapbox.com/api-playground/#/directions/).

URL parameter | Description
--- | ---
`profile` | The routing profile to use. Possible values are `mapbox/driving-traffic`, `mapbox/driving`, `mapbox/walking`, or `mapbox/cycling`.
`coordinates` | A semicolon-separated list of `{longitude},{latitude}` coordinate pairs to visit in order. There can be between 2 and 25 coordinates for most requests, or up to 3 coordinates for `driving-traffic` requests.

You can further refine the results from this endpoint with the following optional parameters:

Query parameter | Description
--- | ---
`alternatives`<br /> (optional) | Whether to try to return alternative routes (`true`) or not (`false`, default). An alternative route is a route that is significantly different than the fastest route, but also still reasonably fast. Such a route does not exist in all circumstances. Up to 2 alternatives may be returned.
`annotations`<br /> (optional) | Return additional metadata along the route. Possible values are: `duration`, `distance`, `speed`, and `congestion`. You can include several annotations as a comma-separated list. See the [route leg object](#routeleg-object) for more details on what is included with annotations. Must be used in conjunction with `overview=full`.
`approaches`<br /> (optional) | A semicolon-separated list indicating the side of the road from which to approach waypoints in a requested route. Accepts `unrestricted` (default, route can arrive at the waypoint from either side of the road) or `curb` (route will arrive at the waypoint on the `driving_side` of the region). If provided, the number of approaches must be the same as the number of waypoints. However, you can skip a coordinate and show its position in the list with the `;` separator. Must be used in combination with `steps=true`.
`banner_instructions`<br /> (optional) | Whether to return banner objects associated with the route steps (`true`) or not (`false`, default). Must be used in conjunction with `steps=true`.
`bearings`<br /> (optional) | Influences the direction in which a route *starts* from a waypoint. Used to filter the road segment the waypoint will be placed on by direction. This is useful for making sure the new routes of rerouted vehicles continue traveling in their current direction. A request that does this would provide bearing and radius values for the first waypoint and leave the remaining values empty. Must be used in conjunction with the `radiuses` parameter. Takes 2 comma-separated values per waypoint: an angle clockwise from true north between 0 and 360, and the range of degrees by which the angle can deviate (recommended value is 45° or 90°), formatted as `{angle, degrees}`. If provided, the list of bearings must be the same length as the list of coordinates. However, you can skip a coordinate and show its position in the list with the `;` separator.
`continue_straight`<br /> (optional) | Sets the allowed direction of travel when departing intermediate waypoints. If `true`, the route will continue in the same direction of travel. If `false`, the route may continue in the opposite direction of travel. Defaults to `true` for `mapbox/driving` and `false` for `mapbox/walking` and `mapbox/cycling`.
`exclude`<br/> (optional) | Exclude certain road types from routing. The default is to not exclude anything from the profile selected. The following `exclude` flags are available for each profile:<table><th>**Profile**</th><th>**Available excludes**</th><tr><td>`mapbox/driving`</td><td>One of `toll`, `motorway`, or `ferry`</td></tr><tr><td>`mapbox/driving-traffic`</td><td>One of `toll`, `motorway`, or `ferry`</td></tr><tr><td>`mapbox/walking`</td><td>No excludes supported</td></tr><tr><td>`mapbox/cycling`</td><td>`ferry`</td></tr></table>
`geometries`<br /> (optional) | The format of the returned geometry. Allowed values are: `geojson` (as [LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2)), [`polyline`](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) (default, a polyline with precision 5), [`polyline6`](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) (a polyline with precision 6).
`language`<br /> (optional) | The language of returned turn-by-turn text instructions. See [supported languages](#instructions-languages). The default is `en` (English). Must be used in combination with `steps=true`.
`overview`<br /> (optional) | The type of returned overview geometry. Can be `full` (the most detailed geometry available), `simplified` (default, a simplified version of the full geometry), or `false` (no overview geometry).
`radiuses`<br /> (optional) | The maximum distance a coordinate can be moved to snap to the road network in meters. There must be as many radiuses as there are coordinates in the request, each separated by `;`. Values can be any number greater than `0` or the string `unlimited`. A `NoSegment` error is returned if no routable road is found within the radius.
`roundabout_exits`<br /> (optional) | Whether to emit instructions at roundabout exits (`true`) or not (`false`, default). Without this parameter, roundabout maneuvers are given as a single instruction that includes both entering and exiting the roundabout. With `roundabout_exits=true`, this maneuver becomes two instructions, one for entering the roundabout and one for exiting it.
`steps`<br /> (optional) | Whether to return steps and turn-by-turn instructions (`true`) or not (`false`, default).
`voice_instructions`<br /> (optional) | Whether to return [SSML](https://developer.amazon.com/docs/custom-skills/speech-synthesis-markup-language-ssml-reference.html) marked-up text for voice guidance along the route (`true`) or not (`false`, default). Must be used in conjunction with `steps=true`.
`voice_units`<br /> (optional) | Specify which type of units to return in the text for voice instructions. Can be `imperial` (default) or `metric`. Must be used in conjunction with be used in conjunction with `steps=true` and `voice_instructions=true`.
`waypoint_names`<br /> (optional) | A semicolon-separated list of custom names for coordinates used for the arrival instruction in banners and voice instructions. Values can be any string, and the total number of all characters cannot exceed 500. If provided, the list of `waypoint_names` must be the same length as the list of coordinates, but you can skip a coordinate and show its position with the `;` separator.

Unrecognized options in the query string result in an `InvalidInput` error.

**Instructions languages**<a id='instructions-languages'></a>

The table below shows supported language codes used with the `language` parameter for turn-by-turn instructions. The language parameter will automatically be set to `en` (English) if an unsupported language code is used.

Code | Language
--- | ---
`da` | Danish
`de` | German
`en` | English
`eo` | Esperanto
`es` | Spanish
`es-ES` | Spanish (Spain)
`fi` | Finnish
`fr` | French
`he` | Hebrew
`id` | Indonesian
`it` | Italian
`ko` | Korean
`my` | Burmese
`nl` | Dutch
`no` | Norwegian (Bokmål)
`pl` | Polish
`pt-BR` | Portuguese (Brazil)
`pt-PT` | Portuguese (Portugal)
`ro` | Romanian
`ru` | Russian
`sv` | Swedish
`tr` | Turkish
`uk` | Ukrainian
`vi` | Vietnamese
`zh-Hans` | Chinese (Simplified)

#### Example request

```curl
# Request directions with no additional options
curl "https://api.mapbox.com/directions/v5/mapbox/cycling/-122.42,37.78;-77.03,38.91?access_token={your_access_token}"

# Request directions with steps and alternatives
curl "https://api.mapbox.com/directions/v5/mapbox/cycling/-122.42,37.78;-77.03,38.91?steps=true&alternatives=true&access_token={your_access_token}"

# Request directions with radiuses and a polyline6 response through multiple waypoints
curl "https://api.mapbox.com/directions/v5/mapbox/driving/13.43,52.51;13.42,52.5;13.41,52.5?radiuses=40;;100&geometries=polyline6&access_token={your_access_token}"

# Specify bearings and radiuses
curl "https://api.mapbox.com/directions/v5/mapbox/driving/13.43,52.51;13.42,52.5;13.43,52.5?bearings=60,45;;45,45&radiuses=100;100;100&access_token={your_access_token}"

# Request a route that approaches the destination on the curb of the driving side
curl "https://api.mapbox.com/directions/v5/mapbox/driving/13.43,52.51;13.43,52.5?approaches=unrestricted;curb&access_token={your_access_token}"

# Request directions with voice and banner instructions
curl "https://api.mapbox.com/directions/v5/mapbox/cycling/-122.42,37.78;-77.03,38.91?steps=true&voice_instructions=true&banner_instructions=true&voice_units=imperial&access_token={your_access_token}"

# Request directions with voice and banner instructions specifying waypoint_names
curl "https://api.mapbox.com/directions/v5/mapbox/cycling/-122.42,37.78;-77.03,38.91?steps=true&voice_instructions=true&banner_instructions=true&voice_units=imperial&waypoint_names=Home;Work&access_token={your_access_token}"
```

```python
service = Directions()
# The input waypoints are features, typically GeoJSON-like feature dictionaries
origin = {
    'type': 'Feature',
    'properties': {'name': 'Portland, OR'},
    'geometry': {
        'type': 'Point',
        'coordinates': [-122.7282, 45.5801]}}
destination = {
    'type': 'Feature',
    'properties': {'name': 'Bend, OR'},
    'geometry': {
        'type': 'Point',
        'coordinates': [-121.3153, 44.0582]}}
# The directions() method can be called with a list of features and the desired profile
response = service.directions([origin, destination], 'mapbox.driving')
```

```javascript
directionsClient
  .getDirections({
    waypoints: [
      {
        coordinates: [13.4301, 52.5109],
        approach: 'unrestricted'
      },
      {
        coordinates: [13.4265, 52.508]
      },
      {
        coordinates: [13.4194, 52.5072],
        bearing: [100, 60]
      }
    ]
  })
  .send()
  .then(response => {
    const directions = response.body;
  });
```

```bash
# This API is not available through the Mapbox CLI
```

```java
MapboxDirections client = MapboxDirections.builder()
  .accessToken("{your_access_token}")
  .origin(origin)
  .destination(destination)
  .overview(DirectionsCriteria.OVERVIEW_FULL)
  .profile(DirectionsCriteria.PROFILE_CYCLING)
  .build();
```

```objc
NSArray<MBWaypoint *> *waypoints = @[
    [[MBWaypoint alloc] initWithCoordinate:CLLocationCoordinate2DMake(38.9099711, -77.0361122) name:@"Mapbox"],
    [[MBWaypoint alloc] initWithCoordinate:CLLocationCoordinate2DMake(38.8977, -77.0365) name:@"White House"],
];
MBRouteOptions *options = [[MBRouteOptions alloc] initWithWaypoints:waypoints profileIdentifier:MBDirectionsProfileIdentifierCycling];
options.includesSteps = YES;

NSURLSessionDataTask *task = [directions calculateDirectionsWithOptions:options completionHandler:^(NSArray<MBWaypoint *> * _Nullable waypoints, NSArray<MBRoute *> * _Nullable routes, NSError * _Nullable error) {
    if (error) {
        NSLog(@"Error calculating directions: %@", error);
        return;
    }

    MBRoute *route = routes.firstObject;
    MBRouteLeg *leg = route.legs.firstObject;
    if (leg) {
        NSLog(@"Route via %@:", leg);

        NSLengthFormatter *distanceFormatter = [[NSLengthFormatter alloc] init];
        NSString *formattedDistance = [distanceFormatter stringFromMeters:leg.distance];

        NSDateComponentsFormatter *travelTimeFormatter = [[NSDateComponentsFormatter alloc] init];
        travelTimeFormatter.unitsStyle = NSDateComponentsFormatterUnitsStyleShort;
        NSString *formattedTravelTime = [travelTimeFormatter stringFromTimeInterval:route.expectedTravelTime];

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
let waypoints = [
    Waypoint(
      coordinate: CLLocationCoordinate2D(latitude: 38.9099711, longitude: -77.0361122),
      name: "Mapbox"),
    Waypoint(
      coordinate: CLLocationCoordinate2D(latitude: 38.8977, longitude: -77.0365),
      name: "White House"),
]
let options = RouteOptions(waypoints: waypoints, profileIdentifier: .cycling)
options.includesSteps = true

let task = directions.calculate(options) { (waypoints, routes, error) in
    guard error == nil else {
        print("Error calculating directions: \(error!)")
        return
    }

    if let route = routes?.first, let leg = route.legs.first {
        print("Route via \(leg):")

        let distanceFormatter = LengthFormatter()
        let formattedDistance = distanceFormatter.string(fromMeters: route.distance)

        let travelTimeFormatter = DateComponentsFormatter()
        travelTimeFormatter.unitsStyle = .short
        let formattedTravelTime = travelTimeFormatter.string(from: route.expectedTravelTime)

        print("Distance: \(formattedDistance); ETA: \(formattedTravelTime!)")

        for step in leg.steps {
            print("\(step.instructions)")
            let formattedDistance = distanceFormatter.string(fromMeters: step.distance)
            print("— \(formattedDistance) —")
        }
    }
}
```

### Directions response object

The response to a Directions API request is a JSON object that contains the following properties:

Property | Description
--- | ---
`code` | A string indicating the state of the response. This is a different code than the HTTP status code. On normal valid responses, the value will be `Ok`. For other responses, see the [Directions API errors table](#directions-api-errors).
`waypoints` | An array of [waypoint](#waypoint-object) objects. Each waypoint is an input coordinate snapped to the road and path network. The waypoints appear in the array in the order of the input coordinates.
`routes` | An array of [route](#route-object) objects ordered by descending recommendation rank. The response object may contain at most 2 routes.

#### Example response object

```json
{
  "routes": [
    {
      "geometry": "mnn_Ick}pAfBiF`CzA",
      "legs": [
        {
          "summary": "Köpenicker Straße, Engeldamm",
          "weight": 44.4,
          "duration": 26.2,
          "steps": [
            {
              "intersections": [
                {
                  "out": 0,
                  "entry": [ true ],
                  "bearings": [ 125 ],
                  "location": [ 13.426579, 52.508068 ]
                },
                {
                  "out": 1,
                  "in": 2,
                  "entry": [ true, true, false ],
                  "bearings": [ 30, 120, 300 ],
                  "location": [ 13.426688, 52.508022 ]
                }
              ],
              "driving_side": "right",
              "geometry": "mnn_Ick}pAHUlAqDNa@",
              "mode": "driving",
              "maneuver": {
                "bearing_after": 125,
                "bearing_before": 0,
                "location": [ 13.426579, 52.508068 ],
                "modifier": "right",
                "type": "depart",
                "instruction": "Head southeast on Köpenicker Straße (L 1066)"
              },
              "ref": "L 1066",
              "weight": 35.9,
              "duration": 17.7,
              "name": "Köpenicker Straße (L 1066)",
              "distance": 98.1,
              "voiceInstructions": [
                {
                  "distanceAlongGeometry": 98.1,
                  "announcement": "Head southeast on Köpenicker Straße (L 1066), then turn right onto Engeldamm",
                  "ssmlAnnouncement": "<speak><amazon:effect name=\"drc\"><prosody rate=\"1.08\">Head southeast on Köpenicker Straße (L <say-as interpret-as=\"address\">1066</say-as>), then turn right onto Engeldamm</prosody></amazon:effect></speak>"
                },
                {
                  "distanceAlongGeometry": 83.1,
                  "announcement": "Turn right onto Engeldamm, then you will arrive at your destination",
                  "ssmlAnnouncement": "<speak><amazon:effect name=\"drc\"><prosody rate=\"1.08\">Turn right onto Engeldamm, then you will arrive at your destination</prosody></amazon:effect></speak>"
                }
              ],
              "bannerInstructions": [
                {
                  "distanceAlongGeometry": 98.1,
                  "primary": {
                    "text": "Engeldamm",
                    "components": [ { "text": "Engeldamm" } ],
                    "type": "turn",
                    "modifier": "right"
                  },
                  "secondary": null,
                  "then": null
                }
              ]
            },
            {
              "intersections": [
                {
                  "out": 2,
                  "in": 3,
                  "entry": [ false, true, true, false ],
                  "bearings": [ 30, 120, 210, 300 ],
                  "location": [ 13.427752, 52.50755 ]
                }
              ],
              "driving_side": "right",
              "geometry": "ekn_Imr}pARL\\T^RHDd@\\",
              "mode": "driving",
              "maneuver": {
                "bearing_after": 202,
                "bearing_before": 125,
                "location": [ 13.427752, 52.50755 ],
                "modifier": "right",
                "type": "turn",
                "instruction": "Turn right onto Engeldamm"
              },
              "weight": 8.5,
              "duration": 8.5,
              "name": "Engeldamm",
              "distance": 78.6,
              "voiceInstructions": [
                {
                  "distanceAlongGeometry": 27.7,
                  "announcement": "You have arrived at your destination",
                  "ssmlAnnouncement": "<speak><amazon:effect name=\"drc\"><prosody rate=\"1.08\">You have arrived at your destination</prosody></amazon:effect></speak>"
                }
              ],
              "bannerInstructions": [
                {
                  "distanceAlongGeometry": 78.6,
                  "primary": {
                    "text": "You will arrive at your destination",
                    "components": [ { "text": "You will arrive at your destination" } ],
                    "type": "arrive",
                    "modifier": "straight"
                  },
                  "secondary": {
                    "text": "Engeldamm",
                    "components": [ { "text": "Engeldamm" } ],
                    "type": "arrive",
                    "modifier": "straight"
                  }
                },
                {
                  "distanceAlongGeometry": 15,
                  "primary": {
                    "text": "You have arrived at your destination",
                    "components": [ { "text": "You have arrived at your destination" } ],
                    "type": "arrive",
                    "modifier": "straight"
                  },
                  "secondary": {
                    "text": "Engeldamm",
                    "components": [ { "text": "Engeldamm" } ]
                  }
                }
              ]
            },
            {
              "intersections": [
                {
                  "in": 0,
                  "entry": [ true ],
                  "bearings": [ 25 ],
                  "location": [ 13.427292, 52.506902 ]
                }
              ],
              "driving_side": "right",
              "geometry": "cgn_Iqo}pA",
              "mode": "driving",
              "maneuver": {
                "bearing_after": 0,
                "bearing_before": 205,
                "location": [ 13.427292, 52.506902 ],
                "type": "arrive",
                "instruction": "You have arrived at your destination"
              },
              "weight": 0,
              "duration": 0,
              "name": "Engeldamm",
              "distance": 0,
              "voiceInstructions": [ ],
              "bannerInstructions": [ ]
            }
          ],
          "distance": 176.7
        }
      ],
      "weight_name": "routability",
      "weight": 44.4,
      "duration": 26.2,
      "distance": 176.7
    }
  ],
  "waypoints": [
    {
      "name": "Köpenicker Straße",
      "location": [ 13.426579, 52.508068 ]
    },
    {
      "name": "Engeldamm",
      "location": [ 13.427292, 52.506902 ]
    }
  ],
  "code": "Ok",
  "uuid": "cjd51uqn5005447p8nte2zc4w"
}
```

### Waypoint object

The response body of a Directions API query contains a **waypoint object**, the input coordinates snapped to the roads network. A waypoint object contains the following properties:

Property | Description
--- | ---
`name` | A string with the name of the road or path to which the input coordinate has been snapped.
`location` | An array containing the `[longitude, latitude]` of the snapped coordinate.

#### Example waypoint object

```json
{
    "name": "Kirkjubøarvegur",
    "location": [ -6.80897, 62.000075 ]
}
```

### Route object

The response body of a Directions API query also contains an array of **route objects**. A route object describes a route through multiple waypoints. A route object contains the following properties:

Property | Description
--- | ---
`duration` | A float indicating the estimated travel time through the waypoints in seconds.
`distance` | A float indicating the distance traveled through the waypoints in meters.
`weight_name` | A string indicating which weight was used. The default is `routability`, which is duration-based, with additional penalties for less desirable maneuvers.
`weight` | A float indicating the weight in units described by `weight_name`.
`geometry` | Depending on the `geometries` query parameter, this is either a [GeoJSON LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2) or a [Polyline string](https://developers.google.com/maps/documentation/utilities/polylinealgorithm). Depending on the `overview` query parameter, this is the complete route geometry (`full`), a simplified geometry to the zoom level at which the route can be displayed in full (`simplified`), or is not included (`false`).
`legs` | An array of [route leg](#routeleg-object) objects.
`voiceLocale` | A string of the locale used for voice instructions. Defaults to `en` (English). Can be any [accepted instruction language](#instructions-languages). `voiceLocale` is only present in the response when `voice_instructions=true`.

#### Example route object

```json
{
    "duration": 88.4,
    "distance": 830.4,
    "weight": 88.4,
    "weight_name": "routability",
    "geometry": "oklyJ`{ph@yBuY_F{^_FxJoBrBs@d@mAT",
    "legs": [ ],
    "voiceLocale": "en"
}
```

### Route leg object

A route object contains a nested **route leg** object for each leg of the journey, which is one fewer than the number of input coordinates. A route leg object contains the following properties:

Property | Description
--- | ---
`distance` | A number indicating the distance traveled between waypoints in meters.
`duration` | A number indicating the estimated travel time between waypoints in seconds.
`steps` | Depending on the optional `steps` parameter, either an array of [route step](#routestep-object) objects (`steps=true`) or an empty array (`steps=false`, default).
`summary` | A string summarizing the route.
`annotation` | An annotations object that contains additional details about each line segment along the route geometry. Each entry in an annotations field corresponds to a coordinate along the route geometry. See the following table for more information about the `annotation` object:

`annotation` | Description
--- | ---
`distance` | The distance between each pair of coordinates in meters.
`duration` | The duration between each pair of coordinates in seconds.
`speed` | The speed between each pair of coordinates in meters per second.
`congestion` | The level of congestion, described as `severe`, `heavy`, `moderate`, `low` or `unknown`, between each entry in the array of coordinate pairs in the route leg. For any profile other than `mapbox/driving-traffic` a list of `unknown`s will be returned. A list of `unknown`s will also be returned if the route is very long.

#### Example route leg object

```json
{
    "annotation": {
        "distance": [
            4.294596842089401,
            5.051172053200946,
            5.533254065167979,
            6.576513793849532,
            7.4449640160938015,
            8.468757534990829,
            15.202780313562714,
            7.056346577326572
        ],
        "duration": [
            1,
            1.2,
            2,
            1.6,
            1.8,
            2,
            3.6,
            1.7
        ],
        "speed": [
            4.3,
            4.2,
            2.8,
            4.1,
            4.1,
            4.2,
            4.2,
            4.2
        ],
        "congestion": [
            "low",
            "moderate",
            "moderate",
            "moderate",
            "heavy",
            "heavy",
            "heavy",
            "heavy"
        ]
    },
    "duration": 14.3,
    "weight": 14.3,
    "distance": 53.4,
    "steps": [],
    "summary": ""
}
```

### Route step object

In a route leg object, a nested **route step** object includes one [step maneuver](#stepmaneuver-object) object as well as information about travel to the following route step:

Property | Description
--- | ---
`maneuver` | One [step maneuver](#stepmaneuver-object) object.
`distance` | A number indicating the distance traveled in meters from the maneuver to the next route step.
`duration` | A number indicating the estimated time traveled in seconds from the maneuver to the next route step.
`geometry` | Depending on the `geometries` parameter, this is a [GeoJSON LineString](https://tools.ietf.org/html/rfc7946#appendix-A.2) or a [Polyline string](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) representing the full route geometry from this route step to the next route step.
`name` | A string with the name of the road or path that forms part of the route step.
`ref` | Any [road designations](https://en.wikipedia.org/wiki/Road_designation_or_abbreviation) associated with the road or path leading from this step’s maneuver to the next step’s maneuver. If [multiple road designations](https://en.wikipedia.org/wiki/Concurrency_%28road%29) are associated with the road, they are separated by semicolons. Typically consists of an alphabetic network code (identifying the road type or numbering system), a space or hyphen, and a [route number](https://en.wikipedia.org/wiki/Route_number). Optionally included, if data is available. <br>**Note:** A network code is not necessarily globally unique, and should not be treated as though it is. A route number may not uniquely identify a road within a given network.
`destinations` | A string with the destinations of the road or path along which the travel proceeds. Optionally included, if data is available.
`exits` | A string with the exit numbers or names of the road or path. Optionally included, if data is available.
`driving_side` | The legal driving side at the location for this step. Either `left` or `right`.
`mode` | A string indicating the mode of transportation. <table><tr><th>Profile</th><th>Possible values</th></tr><tr><td>`mapbox/driving` </td><td>`driving`, `ferry`, `unaccessible`</td></tr><tr><td>`mapbox/walking`</td><td>`walking`, `ferry`, `unaccessible`</td></tr><tr><td>`mapbox/cycling`</td><td>`cycling`, `walking`, `ferry`, `train`, `unaccessible`</td></tr></table>
`pronunciation` | A string containing an [IPA](https://en.wikipedia.org/wiki/International_Phonetic_Alphabet) phonetic transcription indicating how to pronounce the name in the `name` property. Omitted if pronunciation data is not available for the step.
`intersections` | An array of objects representing all the intersections along the step. See the following table for more information on the `intersections` array:


`intersections` | Description
--- | ---
`location` | A `[longitude, latitude]` pair describing the location of the turn.
`bearings` | A list of bearing values that are available at the intersection. The bearings describe all available roads at the intersection.
`classes` | An array of strings signifying the classes of the road exiting the intersection.  <table><tr><th>Possible values</th><th>Description</th></tr><tr><td>`toll`</td><td>Road continues on a toll road</td></tr><tr><td>`ferry`</td><td>Road continues on a ferry</td></tr><tr><td>`restricted`</td><td>Road continues on with access restrictions</td></tr><tr><td>`motorway`</td><td>Road continues on a motorway</td></tr><tr><td>`tunnel`</td><td>Road continues in a tunnel</td></tr></table>
`entry` | A list of entry flags, corresponding 1:1 to `bearings`. If `true`, indicates that the respective road could be entered on a valid route. If `false`, the turn onto the respective road would violate a restriction.
`in` | The index in the `bearings` and `entry` arrays. Used to calculate the bearing before the turn. Namely, the clockwise angle from true north to the direction of travel before the maneuver/passing the intersection. To get the bearing in the direction of driving, the bearing has to be rotated by a value of 180. The value is not supplied for departure maneuvers.
`out` | The index in the `bearings` and `entry` arrays. Used to extract the bearing after the turn. Namely, the clockwise angle from true north to the direction of travel after the maneuver/passing the intersection. The value is not supplied for arrival maneuvers.
`lanes` | An array of [lane](#lane-object) objects that represent the available turn lanes at the intersection. If no lane information is available for an intersection, the `lanes` property will not be present.

#### Example route step object

```json
{
    "intersections": [
      {
        "out": 1,
        "location": [ 13.424671, 52.508812 ],
        "bearings": [ 120, 210, 300 ],
        "entry": [ false, true, true ],
        "in": 0,
        "lanes": [
          {
            "valid": true,
            "indications": [ "left" ]
          },
          {
            "valid": true,
            "indications": [ "straight" ]
          },
          {
            "valid": false,
            "indications": [ "right" ]
          }
        ]
      }
    ],
    "geometry": "asn_Ie_}pAdKxG",
    "maneuver": {
      "bearing_after": 202,
      "type": "turn",
      "modifier": "left",
      "bearing_before": 299,
      "location": [ 13.424671, 52.508812 ],
      "instruction": "Turn left onto Adalbertstraße"
    },
    "duration": 59.1,
    "distance": 236.9,
    "driving_side": "right",
    "weight": 59.1,
    "name": "Adalbertstraße",
    "mode": "driving"
}
```

### Step maneuver object

A route step object contains a nested **step maneuver** object, which contains the following properties:

Property          | Description
------------------| ------------------
`bearing_before`  | A number between `0` and `360` indicating the clockwise angle from true north to the direction of travel immediately _before_ the maneuver.
`bearing_after`   | A number between `0` and `360` indicating the clockwise angle from true north to the direction of travel immediately _after_ the maneuver.
`instruction`     | A human-readable instruction of how to execute the returned maneuver.
`location`        | An array of `[longitude, latitude]` coordinates for the point of the maneuver.
`modifier`        | An optional string indicating the direction change of the maneuver. The meaning of each `modifier` depends on the `type` property. <table><tr><th>Possible values</th><th>Description</th></tr><tr><td>`uturn`</td><td>Indicates a reversal of direction. `type` can be `turn` or `continue` when staying on the same road.</td></tr><tr><td>`sharp right`</td><td>A sharp right turn.</td></tr><tr><td>`right`</td><td>A normal turn to the right.</td></tr><tr><td>`slight right`</td><td>A slight turn to the right.</td></tr><tr><td>`straight`</td><td>No relevant change in direction.</td></tr><tr><td>`slight left`</td><td>A slight turn to the left.</td></tr><tr><td>`left`</td><td>A normal turn to the left.</td></tr><tr><td>`sharp left`</td><td>A sharp turn to the left.</td></tr></table>
`type`            | A string indicating the type of maneuver. See the full list of maneuver types  in the [maneuver types table](#maneuver-types).

- If no `modifier` is provided, the `type` of maneuvers is limited to `depart` and `arrive`.
- If the source or target location is close enough to the `depart` or `arrive` location, no `modifier` will be given.

**Maneuver types**<a id='maneuver-types'></a>

`type` | Description
 --- | ---
`turn` | A basic turn in the direction of the modifier.
`new name` | The road name changes (after a mandatory turn).
`depart` | Indicates departure from a leg. The `modifier` value indicates the position of the departure point in relation to the current direction of travel.
`arrive` | Indicates arrival to a destination of a leg. The `modifier` value indicates the position of the arrival point in relation to the current direction of travel.
`merge` | Merge onto a street.
`on ramp` | Take a ramp to enter a highway.
`off ramp` | Take a ramp to exit a highway.
`fork` | Take the left or right side of a fork.
`end of road` | Road ends in a T intersection.
`continue` | Continue on a street after a turn.
`roundabout` | Traverse roundabout. Has an additional property `exit` in the [route step](#routestep-object) that contains the exit number. The `modifier` specifies the direction of entering the roundabout.
`rotary` | A traffic circle. While very similar to a larger version of a roundabout, it does not necessarily follow roundabout rules for right of way. It can offer `rotary_name` parameters, `rotary_pronunciation` parameters, or both, located in the [route step](#routestep-object) object in addition to the `exit` property.
`roundabout turn` | A small roundabout that is treated as an intersection.
`notification` | Indicates a change of driving conditions, for example changing the `mode` from `driving` to `ferry`.
`exit roundabout` | Indicates the exit maneuver from a roundabout. Will not appear in results unless you supply the `roundabout_exits=true` query parameter in the request.
`exit rotary` | Indicates the exit maneuver from a rotary. Will not appear in results unless you supply the `roundabout_exits=true` query parameter in the request.

**Note:** New properties (potentially depending on `type`) may be introduced in the future without an API version change.

#### Example step maneuver object

```json
{
    "bearing_before": 299,
    "bearing_after": 202,
    "type": "turn",
    "modifier": "left",
    "location": [ 13.424671, 52.508812 ],
    "instruction": "Turn left onto Adalbertstraße"
}
```

### Lane object

A route step object contains a nested **lane object**. The lane object describes the available turn lanes at an intersection. Lanes are provided in their order on the street, from left to right.

Property | Description
--- | ---
 `valid` | Indicates whether a lane can be taken to complete the maneuver (`true`) or not (`false`). For instance, if the lane array has four objects and the first two are valid, the driver can take either of the left lanes and stay on the route.
 `indications` | An array of signs for each turn lane. There can be multiple signs. For example, a turn lane can have a sign with an arrow pointing left and another sign with an arrow pointing straight.

#### Example lane object

```json
{
    "valid": true,
    "indications": [
        "left"
    ]
}
```

### Voice instruction object

A route step object contains a nested **voice instruction object** if the optional `voice_instructions=true` query parameter is present. The voice instruction object contains the text that should be announced, along with how far from the maneuver it should be emitted. The voice instructions are children of the route step during which they should be spoken, but they refer to the maneuver in the _following_ step.

Property | Description
--- | ---
`distanceAlongGeometry` | A float indicating how far from the upcoming maneuver the voice instruction should begin in meters.
`announcement` | A string containing the text of the verbal instruction.
`ssmlAnnouncement` | A string with SSML markup for proper text and pronunciation. This property is designed for use with [Amazon Polly](https://aws.amazon.com/polly/). The SSML tags may not work with other text-to-speech engines.

#### Example voice instruction object

```json
{
    "distanceAlongGeometry": 375.7,
    "announcement": "In a quarter mile, take the ramp on the right towards I 495: Baltimore",
    "ssmlAnnouncement": "<speak><amazon:effect name=\"drc\"><prosody rate=\"1.08\">In a quarter mile, take the ramp on the right towards <say-as interpret-as=\"address\">I-495</say-as>: Baltimore</prosody></amazon:effect></speak>"
}
```

### Banner instruction object

A route step object contains a nested **banner instruction object** if the optional `banner_instructions=true` query parameter is present. The banner instruction object contains the contents of a banner that should be displayed as added visual guidance for a route. The banner instructions are children of the route steps during which they should be displayed, but they refer to the maneuver in the _following_ step.

Property | Description
--- | ---
`distanceAlongGeometry` | A float indicating how far from the upcoming maneuver the banner instruction should begin being displayed in meters. Only one banner should be displayed at a time.
`primary` | The most important content to display to the user. This text is larger and at the top.
`secondary` | Additional content useful for visual guidance. This text is slightly smaller and below `primary`. Can be `null`.
`sub` | Additional information that is included if the driver needs to be notified about something. Can include information about the _next_ maneuver (the one after the upcoming one) if the step is short. If lane information is available, that takes precedence over information about the _next_ maneuver.

Each of the different banner types (`primary`, `secondary`, and `sub`) contains the following properties:

Property | Description
--- | ---
`text` | A string that contains all the text that should be displayed.
`type` | The type of maneuver. May be used in combination with the `modifier` (and, if it is a roundabout, the `degrees`) to for an icon to display. Possible values: `turn`, `merge`, `depart`, `arrive`, `fork`, `off ramp`, and `roundabout`.
`modifier`<br>(optional) | The modifier for the maneuver. Can be used in combination with the `type` (and, if it is a roundabout, the `degrees`) to for an icon to display. <table><tr><th>Possible values</th><th>Description</th></tr><tr><td>`uturn`</td><td>Indicates a reversal of direction</td></tr><tr><td>`sharp right`</td><td>A sharp right turn</td></tr><tr><td>`right`</td><td>A normal turn to the right</td></tr><tr><td>`slight right`</td><td>A slight turn to the right</td></tr><tr><td>`straight`</td><td>No relevant change in direction</td></tr><tr><td>`slight left`</td><td>A slight turn to the left</td></tr><tr><td>`left`</td><td>A normal turn to the left</td></tr><tr><td>`sharp left`</td><td>A sharp turn to the left</td></tr></table>
`degrees`<br>(optional) | The degrees at which you will be exiting a roundabout, assuming `180` indicates going straight through the roundabout.
`driving_side`<br>(optional) | A string representing which side the of the street people drive on in that location. Can be `left` or `right`.
`components` | Objects that, together, make up what should be displayed in the banner. Includes additional information intended to be used to aid in visual layout. A component can contain the following properties:

`components` | Description
--- | ---
`type` | A string with more context about the component that may help in visual markup and display choices. If the type of the component is unknown, it should be treated as text. <table><tr><th>Possible values</th><th>Description</th></tr><tr><td>`text`</td><td>Default. Indicates the text is part of the instructions and no other type.</td></tr><tr><td>`icon` </td><td>This is text that can be replaced by an imageBaseURL icon.</td></tr><tr><td>`delimiter`</td><td>This is text that can be dropped, and should be dropped if you are rendering icons.</td></tr><tr><td>`exit-number`</td><td>Indicates the exit number for the maneuver.</td></tr><tr><td>`exit`</td><td>Provides the the word for _exit_ in the local language.</td></tr><tr><td>`lane`</td><td>Indicates which lanes can be used to complete the maneuver.</td></tr></table> The introduction of new types is not considered a breaking change.
`text` | The sub-string of the `text` of the parent objects that may have additional context associated with it
`abbr`<br>(optional) | The abbreviated form of `text`. If this is present, there will also be an `abbr_priority` value. For an example of using `abbr` and `abbr_priority`, see the [abbreviation examples](#abbreviation-examples) table.
`abbr_priority`<br>(optional) | An integer indicating the order in which the abbreviation `abbr` should be used in place of `text`. The highest priority is `0`, while a higher integer value means it should have a lower priority. There are no gaps in integer values. Multiple components can have the same `abbr_priority`. When this happens, all `components` with the same `abbr_priority` should be abbreviated at the same time. Finding no larger values of `abbr_priority` means that the string is fully abbreviated.
`imageBaseURL`<br>(optional) | A string pointing to a shield image to use instead of the text.
`directions` | An array indicating which directions you can go from a lane (left, right, or straight). If the value is ['left', 'straight'], the driver can go straight or left from that lane. Present if `components.type` is `lane`.
`active` | A boolean that tells you whether that lane can be used to complete the upcoming maneuver. If multiple lanes are active, then they can all be used to complete the upcoming maneuver. Present if `components.type` is `lane`.

**Abbreviation examples**<a id='abbreviation-examples'></a>

`text`             | `abbr`       | `abbr_priority`
-------------------|--------------|--------------------
North              | N            | 1
Franklin Drive     | Franklin Dr  | 0

Given the `components` in the table above, the possible abbreviations are, in order:
- North Franklin Dr
- N Franklin Dr

#### Example banner instruction object

```json
{
    "distanceAlongGeometry": 100,
    "primary": {
        "type": "turn",
        "modifier": "left",
        "text": "I 495 North / I 95",
        "components": [
          {
            "text": "I 495",
            "imageBaseURL": "https://s3.amazonaws.com/mapbox/shields/v3/i-495",
            "type": "icon"
          },
          {
            "text": "North",
            "type": "text",
            "abbr": "N",
            "abbr_priority": 0
          },
          {
            "text": "/",
            "type": "delimiter"
          },
          {
            "text": "I 95",
            "imageBaseURL": "https://s3.amazonaws.com/mapbox/shields/v3/i-95",
            "type": "icon"
          }
        ]
    },
    "secondary": {
        "type": "turn",
        "modifier": "left",
        "text": "Baltimore / Northern Virginia",
        "components": [
          {
            "text": "Baltimore",
            "type": "text"
          },
          {
            "text": "/",
            "type": "text"
          },
          {
            "text": "Northern Virginia",
            "type": "text"
          }
      ]
    },
    "sub": {
        "text": "",
        "components": [
          {
            "text": "",
            "type": "lane",
            "directions": ["left"],
            "active": true
          },
          {
            "text": "",
            "type": "lane",
            "directions": ["left", "straight"],
            "active": true
          },
          {
            "text": "",
            "type": "lane",
            "directions": ["right"],
            "active": false
          }
        ]
    }
}
```

### Directions API errors

On error, the server responds with different HTTP status codes. For responses with HTTP status codes lower than `500`, the JSON response body includes the `code` property, which may be used by client programs to manage control flow. The response body may also include a `message` property with a human-readable explanation of the error.

If a server error occurs, the HTTP status code will be `500` or higher and the response will not include a `code` property.

Response body `code` | HTTP status code | Description
--- | --- |---
`Ok` | `200` | Normal success case.
`NoRoute` | `200` | There was no route found for the given coordinates. Check for impossible routes (for example, routes over oceans without ferry connections).
`NoSegment` | `200` | No road segment could be matched for coordinates. Check for coordinates that are too far away from a road.
`ProfileNotFound` | `404` | Use a valid profile as described in the [list of routing profiles](#directions).
`InvalidInput` | `422` | The given request was not valid. The `message` key of the response will hold an explanation of the invalid input.
