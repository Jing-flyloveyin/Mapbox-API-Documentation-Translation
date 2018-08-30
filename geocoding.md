## Geocoding

The Mapbox Geocoding API does two things: forward geocoding and reverse geocoding.

Forward geocoding lets you convert location text into geographic
coordinates, turning `2 Lincoln Memorial Circle NW` into `-77.050,38.889`.

Reverse geocoding turns geographic coordinates into place names,
turning `-77.050, 38.889` into `2 Lincoln Memorial Circle NW`. These place names
can vary from specific addresses to states and countries that contain
the given coordinates.

Swift and Objective-C support for Geocoding is provided by the [MapboxGeocoder.swift](https://github.com/mapbox/MapboxGeocoder.swift)
library.

**Restrictions and limits**

To access the Geocoding API, you'll need an access token. Rate limits are enforced per account and vary by plan as detailed below:

| Account level | Rate limit |
| ------------- | ---------- |
| Pay-as-you-go       | 600 requests per minute |
| Commercial       | 600 requests per minute |
| Enterprise    | 2,400 requests per minute |

Exceeding the limits above will result in an `HTTP 429` response. For information on rate limit headers, see [Rate limits](#rate-limits).

Batch geocoding is only available [with an Enterprise plan](https://www.mapbox.com/enterprise/).
On all other plan levels, one geocode is permitted per request.

The results from geocoding with the `mapbox.places` mode [must
  be displayed on a Mapbox map and cannot be stored permanently](https://www.mapbox.com/tos/#%5BYmouYmoq%5D). The `mapbox.places-permanent` mode, available [with an Enterprise plan](https://www.mapbox.com/enterprise/),
  does not have these licensing restrictions.

The [Mapbox Geocoding API coverage map](https://www.mapbox.com/geocoding/#coverage) lists the types of geocoding results supported in each area of the world.

Queries are limited to either a total of 20 words and numbers separated by spacing and punctuation _or_ 256 characters.

If you use the optional bounding box parameter to filter results, note that the bounding box cannot cross the 180th meridian.

```objc
@import MapboxGeocoder;

MBGeocoder *geocoder = [[MBGeocoder alloc] initWithAccessToken:@"<#your access token#>"];
```

```objc
@import MapboxGeocoder;

// If you’ve already set `MGLMapboxAccessToken` in your Info.plist:
MBGeocoder *geocoder = [MBGeocoder sharedGeocoder];
```

```swift
import MapboxGeocoder

let geocoder = Geocoder(accessToken: "<#your access token#>")
```

```swift
import MapboxGeocoder

// If you’ve already set `MGLMapboxAccessToken` in your Info.plist:
let geocoder = Geocoder.shared
```

```python
from mapbox import Geocoder
```

```javascript
const mbxGeocoding = require('@mapbox/mapbox-sdk/services/geocoding');
const geocodingClient = mbxGeocoding({ accessToken: '{your_access_token}' });
```

### Request format

Both geocoding and reverse geocoding requests have the same basic format. Since
the `{query}` parameter can contain any value, it should be URL-encoded UTF-8.

```endpoint
GET /geocoding/v5/{mode}/{query}.json
```

URL Parameter | Description
--- | ---
`query` | A location. This will be a place name for forward geocoding or a coordinate pair (longitude, latitude) for reverse geocoding.
`mode` | Either `mapbox.places` for ephemeral geocoding, or `mapbox.places-permanent` for storing results and batch geocoding.

Query Parameter | Description
----------|------------
`country` <br /> (optional) | Limit results to one or more countries. Options are [ISO 3166 alpha 2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country codes separated by commas.
`proximity`<br /> (optional) | Bias local results based on a provided location. Options are `longitude,latitude` coordinates.
`types`<br /> (optional) | Filter results by one or more feature types. Options are `country`, `region`, `postcode`, `district`, `place`, `locality`, `neighborhood`, `address`, `poi`, and `poi.landmark`. Note that `poi.landmark` returns a subset of the results returned by `poi`. Multiple options can be comma-separated.
`autocomplete`<br /> (optional) | _Forward geocoding only._ Return autocomplete results or not. Options are `true` or `false` and the default is `true`.
`bbox`<br /> (optional) | _Forward geocoding only._ Limit results to a bounding box. Options are in the format `minX,minY,maxX,maxY`.
`limit`<br /> (optional) | Limit the number of results returned. The default is `5` for forward geocoding and `1` for reverse geocoding.
`language` <br /> (optional) | Specify the language to use for response text and, for forward geocoding, query result weighting. Options are [IETF language tags](https://en.wikipedia.org/wiki/IETF_language_tag) comprised of a mandatory [ISO 639-1 language code](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) and optionally one or more IETF subtags for country or script. More than one value can also be specified, separated by commas.

The `{proximity}` parameter biases search results within 5 miles of a specific
location given in `{longitude},{latitude}` coordinates. Results will not be
filtered by location or ordered by distance, but location will be considered
along with textual relevance and other criteria when scoring and sorting
results.

The `{autocomplete}` parameter controls whether autocomplete results
are included. Autocomplete results can partially match the query. For example,
searching for `washingto` could include `washington` even though only
the prefix matches. Autocomplete is useful for offering fast, type-ahead
results in user interfaces. If your queries represent complete addresses or
place names, you can disable this behavior and exclude partial matches by
setting the `{autocomplete}` parameter to `false`.

The `{limit}` parameter specifies the *maximum* number of results to
return. For forward geocoding, the default is `5` and the maximum is
`10`. For reverse geocoding, the default is `1` and the maximum is `5`.
If a `limit` other than `1` is used for reverse geocoding, a single `types`
option must also be specified.

The `{language}` parameter specifies the desired response language for user
queries. For forward geocodes, results that match the requested language are
favored over results in other languages. If more than one language tag is
supplied, text in all requested languages will be returned. For forward
geocodes with more than one language tag, only the first language will be used
to weight results.

Any valid IETF language tag can be submitted, and a best effort will be made to
return results in the requested language or languages, falling back first to
similar and then to common languages in the event that text is not available in
the requested language. In the event a fallback language is used, the `language`
field will have a different value than the one requested.

Translation availability varies by language and region:

**Global coverage:** _these languages are almost always present for `country`, `region`, and prominent `place` features_

Language | |
--- | --- | ---
`de` German  | `en` English | `fr` French
`it` Italian | `nl` Dutch   |

-------------

**Local coverage:** _these languages may lack global coverage but are almost always present for `country`, `region`, and prominent `place` features where they are widely used_

Language | |
--- | --- | ---
`bs` Bosnian  | `bg` Bulgarian | `ca` Catalan
`cs` Czech    | `da` Danish    | `el` Greek
`et` Estonian | `fi` Finnish   | `ka` Georgian
`he` Hebrew   | `hu` Hungarian | `is` Icelandic
`ja` Japanese | `ko` Korean    | `lt` Lithuanian
`lv` Latvian  | `mn` Mongolian | `nb` Norwegian Bokmål
`pl` Polish   | `pt` Portuguese| `ro` Romanian
`ru` Russian  | `sk` Slovak    | `sl` Slovenian
`sr` Serbian  | `sv` Swedish   | `tl` Tagalog
`th` Thai     | `uk` Ukrainian | `zh-Hans` Simplified Chinese
`zh-Hant` Traditional Chinese | |

-------------

**Limited coverage:** _these languages are sometimes present but coverage tends to be inconsistent or geographically constrained_

Language | |
--- | --- | ---
`ar` Arabic  | `es` Spanish | `fa` Persian
`kk` Kazakh  | `ms` Malay   | `sq` Albanian
`tr` Turkish | `uz` Uzbek   | `vi` Vietnamese

### Response object

Because Mapbox's geocoding data is constantly updated and improved, the values of properties in the response object are not guaranteed and may change within the same version of the API. Properties may be added to but will not be removed from the response within the same version. Geocoding results are returned
in [Carmen GeoJSON](https://github.com/mapbox/carmen/blob/master/carmen-geojson.md) format.

Property | Description
----------|----------
`type` | `"Feature Collection"`, a GeoJSON type from the [GeoJSON specification](https://tools.ietf.org/html/rfc7946).
`query` | An array of space and punctuation-separated strings from the original query.
`features` | An array of feature objects.
`attribution` | A string attributing the results of the Mapbox Geocoding API to Mapbox and links to Mapbox's terms of service and data sources.

**Feature object**

Each feature object in the `"features"` array may have the properties described below. Forward geocodes return features ordered by `relevance`. Reverse geocodes return features in order of index hierarchy, from most specific features to least specific features that overlap the queried coordinates.

Property | Description
----------|----------
`id` | A string feature id in the form `{type}.{id}` where `{type}` is the lowest hierarchy feature in the `place_type` field. The `{id}` suffix of the feature id is unstable and may change within versions.
`type` | `"Feature"`, a GeoJSON type from the [GeoJSON specification](https://tools.ietf.org/html/rfc7946).
`place_type` | An array of feature types describing the feature. Options are `country`, `region`, `postcode`, `district`, `place`, `locality`, `neighborhood`, `address`, `poi`, and `poi.landmark`. Most features have only one type, but if the feature has multiple types, all applicable types will be listed in the array. (For example, Vatican City is a `country`, `region`, and `place`.)
`relevance` | A numerical score from 0 (least relevant) to 0.99 (most relevant) measuring how well each returned feature matches the query. You can use the `relevance` property to remove results that don't fully match the query.
`address` <br /> (optional) | A string of the house number for the returned `address` feature. Note that unlike the `address` property for `poi` features, this property is outside the `properties` object.
`properties` | An object describing the feature. The property object is unstable and only [Carmen GeoJSON](https://github.com/mapbox/carmen/blob/master/carmen-geojson.md) properties are guaranteed. Your implementation should check for the presence of these values in a response before it attempts to use them.
    `properties.address`<br /> (optional) | A string of the full street address for the returned `poi` feature. Note that unlike the `address` property for `address` features, this property is inside the `properties` object.
    `properties.category`<br /> (optional) | A string of comma-separated categories for the returned `poi` feature.
    `properties.tel`<br /> (optional) | A formatted string of the telephone number for the returned `poi` feature.
    `properties.maki`<br /> (optional) | The name of a suggested [Maki](https://www.mapbox.com/maki-icons/) icon to visualize a `poi` feature based on its `category`.
    `properties.landmark`<br /> (optional) | A boolean value indicating whether a `poi` feature is a landmark. Landmarks are particularly notable or long-lived features like schools, parks, museums and places of worship.
    `properties.wikidata`<br /> (optional) | The [Wikidata](https://wikidata.org) identifier for the returned feature.
    `properties.short_code`<br /> (optional) | The [ISO 3166-1](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country and [ISO 3166-2](https://en.wikipedia.org/wiki/ISO_3166-2) region code for the returned feature.
`text` | A string representing the feature in the requested language, if specified.
`place_name` | A string representing the feature in the requested language, if specified, and its full result hierarchy.
`matching_text`<br /> (optional) | A string analogous to the `text` field that more closely matches the query than results in the specified language. For example, querying "Köln, Germany" with language set to English might return a feature with the `text` "Cologne" and the `matching_text` "Köln".
`matching_place_name`<br /> (optional) | A string analogous to the `place_name` field that more closely matches the query than results in the specified language. For example, querying "Köln, Germany" with language set to English might return a feature with the `place_name` "Cologne, Germany" and a `matching_place_name` of "Köln, North Rhine-Westphalia, Germany".
`text_{language}`<br /> (optional) | A string analogous to the `text` field that matches the query in the requested language. This field is only returned when requesting  multiple languages and will be present for each requested language.
`place_name_{language}`<br /> (optional) | A string analogous to the `place_name` field that matches the query in the requested language. This field is only returned when requesting  multiple languages and will be present for each requested language.
`language` <br /> (optional) | A string of the [IETF language tag](https://en.wikipedia.org/wiki/IETF_language_tag) of the query's primary language.
`language_{language}` <br /> (optional) | A string of the [IETF language tag](https://en.wikipedia.org/wiki/IETF_language_tag) of the query's fallback language. This field is only returned when requesting multiple languages and will be present for each requested language.
`bbox` | An array bounding box in the form [minX,minY,maxX,maxY].
`center` | An array in the form [longitude,latitude] at the center of the specified `bbox`.
`geometry` | An object describing the spatial geometry of the returned feature.
    `geometry.type` | `"Point"`, a GeoJSON type from the [GeoJSON specification](https://tools.ietf.org/html/rfc7946).
    `geometry.coordinates` | An array in the format [longitude,latitude] at the center of the specified `bbox`.
    `geometry.interpolated`<br /> (optional) | A boolean value indicating if an `address` is interpolated along a road network. This field is only present when the feature is interpolated.
`context` | An array representing the hierarchy of encompassing parent features. Each parent feature may include any of the above properties.


```json
{
  "type": "FeatureCollection",
  "query": [
    "los",
    "angeles"
  ],
  "features": [
    {
      "id": "place.9962989141465270",
      "type": "Feature",
      "place_type": [
        "place"
      ],
      "relevance": 0.99,
      "properties": {
        "wikidata": "Q65"
      },
      "text": "Los Angeles",
      "place_name": "Los Angeles, California, United States",
      "bbox": [
        -118.529221009603,
        33.901599990108,
        -118.121099990025,
        34.1612200099034
      ],
      "center": [
        -118.2439,
        34.0544
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          -118.2439,
          34.0544
        ]
      },
      "context": [
        {
          "id": "region.3591",
          "short_code": "US-CA",
          "wikidata": "Q99",
          "text": "California"
        },
        {
          "id": "country.3145",
          "short_code": "us",
          "wikidata": "Q30",
          "text": "United States"
        }
      ]
    },
    {
      "id": "place.10952642230180310",
      "type": "Feature",
      "place_type": [
        "place"
      ],
      "relevance": 0.99,
      "properties": {
        "wikidata": "Q16910"
      },
      "text": "Los Ángeles",
      "place_name": "Los Ángeles, Bío Bío, Chile",
      "bbox": [
        -72.68248,
        -37.663862,
        -72.041277,
        -37.178368
      ],
      "center": [
        -72.35,
        -37.46667
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          -72.35,
          -37.46667
        ]
      },
      "context": [
        {
          "id": "region.3552",
          "short_code": "CL-BI",
          "wikidata": "Q2170",
          "text": "Bío Bío"
        },
        {
          "id": "country.344",
          "short_code": "cl",
          "wikidata": "Q298",
          "text": "Chile"
        }
      ]
    },
    {
      "id": "poi.1611278850983920",
      "type": "Feature",
      "place_type": [
        "poi"
      ],
      "relevance": 0.99,
      "properties": {
        "address": "1 World Way",
        "category": "international airport, airport",
        "tel": "(310) 646-5252",
        "wikidata": "Q8731",
        "landmark": true,
        "maki": "airport"
      },
      "text": "Los Angeles International Airport",
      "place_name": "Los Angeles International Airport, 1 World Way, Los Angeles, California 90045, United States",
      "center": [
        -118.408056,
        33.9425
      ],
      "geometry": {
        "coordinates": [
          -118.408056,
          33.9425
        ],
        "type": "Point"
      },
      "context": [
        {
          "id": "neighborhood.33720",
          "text": "Westchester"
        },
        {
          "id": "postcode.8081932850252730",
          "text": "90045"
        },
        {
          "id": "place.9962989141465270",
          "wikidata": "Q65",
          "text": "Los Angeles"
        },
        {
          "id": "region.3591",
          "short_code": "US-CA",
          "wikidata": "Q99",
          "text": "California"
        },
        {
          "id": "country.3145",
          "short_code": "us",
          "wikidata": "Q30",
          "text": "United States"
        }
      ]
    },
    {
      "id": "neighborhood.2104633",
      "type": "Feature",
      "place_type": [
        "neighborhood"
      ],
      "relevance": 0.99,
      "properties": {},
      "text": "Los Angeles Heights - Keystone",
      "place_name": "Los Angeles Heights - Keystone, San Antonio, Texas 78201, United States",
      "bbox": [
        -98.534942,
        29.453364,
        -98.514652,
        29.485214
      ],
      "center": [
        -98.52,
        29.47
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          -98.52,
          29.47
        ]
      },
      "context": [
        {
          "id": "postcode.7069290925572850",
          "text": "78201"
        },
        {
          "id": "place.7705127234253710",
          "wikidata": "Q975",
          "text": "San Antonio"
        },
        {
          "id": "region.3818",
          "short_code": "US-TX",
          "wikidata": "Q1439",
          "text": "Texas"
        },
        {
          "id": "country.3145",
          "short_code": "us",
          "wikidata": "Q30",
          "text": "United States"
        }
      ]
    },
    {
      "id": "poi.17555628334440500",
      "type": "Feature",
      "place_type": [
        "poi"
      ],
      "relevance": 0.99,
      "properties": {
        "address": "3939 S Figueroa St",
        "category": "stadium, arena",
        "tel": "(213) 747-7111",
        "wikidata": "Q849784",
        "landmark": true,
        "maki": "baseball"
      },
      "text": "Los Angeles Memorial Coliseum",
      "place_name": "Los Angeles Memorial Coliseum, 3939 S Figueroa St, Los Angeles, California 90037, United States",
      "center": [
        -118.287778,
        34.014167
      ],
      "geometry": {
        "coordinates": [
          -118.287778,
          34.014167
        ],
        "type": "Point"
      },
      "context": [
        {
          "id": "neighborhood.293901",
          "text": "South Los Angeles"
        },
        {
          "id": "postcode.15612390715732530",
          "text": "90037"
        },
        {
          "id": "place.9962989141465270",
          "wikidata": "Q65",
          "text": "Los Angeles"
        },
        {
          "id": "region.3591",
          "short_code": "US-CA",
          "wikidata": "Q99",
          "text": "California"
        },
        {
          "id": "country.3145",
          "short_code": "us",
          "wikidata": "Q30",
          "text": "United States"
        }
      ]
    }
  ],
  "attribution": "NOTICE: © 2017 Mapbox and its suppliers. All rights reserved. Use of this data is subject to the Mapbox Terms of Service (https://www.mapbox.com/about/maps/). This response and the information it contains may not be retained."
}
```

### Search for places

This is often called **forward geocoding**. Request feature data that best matches the input `{query}` text. The response includes one or more results ordered by relevance.

This API is used by the [Mapbox.js L.mapbox.geocoderControl](https://www.mapbox.com/mapbox.js/api/v2.3.0/l-mapbox-geocodercontrol/)
and the [mapbox-gl-geocoder](https://github.com/mapbox/mapbox-gl-geocoder) user interfaces.

[Try this in the API Playground](https://www.mapbox.com/api-playground/#/forward-geocoding/).


```endpoint
GET /geocoding/v5/{mode}/{query}.json
```

#### Example request

```curl
$ curl "https://api.mapbox.com/geocoding/v5/mapbox.places/Los%20Angeles.json?access_token={your_access_token}"
```


```javascript
geocodingClient
  .forwardGeocode({
    query: 'Paris, France',
    limit: 2
  })
  .send()
  .then(response => {
    const match = response.body;
  });

// geocoding with proximity
geocodingClient
  .forwardGeocode({
    query: 'Paris, France',
    proximity: [-95.4431142, 33.6875431]
  })
  .send()
  .then(response => {
    const match = response.body;
  });

// geocoding with countries
geocodingClient
  .forwardGeocode({
    query: 'Paris, France',
    countries: ['fr']
  })
  .send()
  .then(response => {
    const match = response.body;
  });

// geocoding with bounding box
geocodingClient
  .forwardGeocode({
    query: 'Paris, France',
    bbox: [2.14, 48.72, 2.55, 48.96]
  })
  .send()
  .then(response => {
    const match = response.body;
  });
```

```python
geocoder = Geocoder()
response = geocoder.forward("200 queen street")
first = response.geojson()['features'][0]
first['place_name']
# '200 Queen St, Saint John, New Brunswick E2L 2X1, Canada'

# Geocoding with proximity
response = geocoder.forward(
    "200 queen street", lon=-66.05, lat=45.27)
first = response.geojson()['features'][0]
first['place_name']
# '200 Queen St, Saint John, New Brunswick E2L 2X1, Canada'
[int(coord) for coord in first['geometry']['coordinates']]
# [-66, 45]

# Geocoding with a bounding box
response = geocoder.forward(
    "washington", bbox=[-78.338320,38.520792,-77.935454,38.864909])
first = response.geojson()['features'][0]
first['place_name']
# 'Washington, Virginia, United States'
[round(coord, 2) for coord in first['geometry']['coordinates']]
# [-78.16, 38.71]

# Limit results to a certain number
response = geocoder.forward(
    "washington", limit=3)
len(response.geojson()['features'])
# 3

# Filter by country code
response = geocoder.forward("200 queen street", country=['us'])
any(['Canada' in f['place_name'] for f in response.geojson()['features']])
# False

# Filter results to only include points of interest
response = geocoder.forward(
    "new york", types=['poi'])
features = response.geojson()['features']
all([f['id'].startswith('poi') for f in features])
# True
```

```curl
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/2+lincoln+memorial+circle+nw.json?access_token={your_access_token}"

# use the country option to limit
# results to Canada and exclude the
# Lincoln Memorial in America
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/2+lincoln+memorial+circle+nw.json?country=ca&access_token={your_access_token}"

# there are many towns named 'Chester', but adding the
# proximity parameter with local coordinates ensures that the town
# of Chester, New Jersey is in the search results
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/chester.json?proximity=-74.70850,40.78375&access_token={your_access_token}"

# search only for countries named Georgia,
# so that we don't find the state in America
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/georgia.json?types=country&access_token={your_access_token}"

# use the bounding box option to limit results for a
# search for "Starbucks" to those within Washington, D.C.
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/starbucks.json?bbox=-77.083056,38.908611,-76.997778,38.959167&access_token={your_access_token}"

# use the limit option to limit the number of
# results to `2`. Even though there are many possible matches
# for `Washington`, this query will only return 2 results.
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/Washington.json?limit=2&access_token={your_access_token}"
```

```bash
mapbox geocode '2 Lincoln Memorial Circle NW' --bbox=[-78.3284,38.6039,-78.0428,38.7841]
```

```java
// Point used for proximity
Point searchLocationPoint = Point.fromLngLat(2.2869, 48.8590);

MapboxGeocoding client = MapboxGeocoding.builder()
  .accessToken("{your_access_token}")
  .proximity(searchLocationPoint)
  .query("Paris, France")
  .build();   
```

```objc
#if !TARGET_OS_TV
@import Contacts;
#endif

MBForwardGeocodeOptions *options = [[MBForwardGeocodeOptions alloc] initWithQuery:@"200 queen street"];

// To refine the search, you can set various properties on the options object.
options.allowedISOCountryCodes = @[@"CA"];
options.focalLocation = [[CLLocation alloc] initWithLatitude:45.3 longitude:-66.1];
options.allowedScopes = MBPlacemarkScopeAddress | MBPlacemarkScopePointOfInterest;

NSURLSessionDataTask *task = [geocoder geocodeWithOptions:options completionHandler:^(NSArray<MBGeocodedPlacemark *> * _Nullable placemarks, NSString * _Nullable attribution, NSError * _Nullable error) {
    MBPlacemark *placemark = placemarks[0];
    NSLog(@"%@", placemark.name);
        // 200 Queen St
    NSLog(@"%@", placemark.qualifiedName);
        // 200 Queen St, Saint John, New Brunswick E2L 2X1, Canada

    CLLocationCoordinate2D coordinate = placemark.location.coordinate;
    NSLog(@"%f, %f", coordinate.latitude, coordinate.longitude);
        // 45.270093, -66.050985

#if !TARGET_OS_TV
    CNPostalAddressFormatter *formatter = [[CNPostalAddressFormatter alloc] init];
    NSLog(@"%@", [formatter stringFromPostalAddress:placemark.postalAddress]);
        // 200 Queen St
        // Saint John New Brunswick E2L 2X1
        // Canada
#endif
}];
```

```swift
#if !os(tvOS)
    import Contacts
#endif

let options = ForwardGeocodeOptions(query: "200 queen street")

// To refine the search, you can set various properties on the options object.
options.allowedISOCountryCodes = ["CA"]
options.focalLocation = CLLocation(latitude: 45.3, longitude: -66.1)
options.allowedScopes = [.address, .pointOfInterest]

let task = geocoder.geocode(options) { (placemarks, attribution, error) in
    guard let placemark = placemarks?.first else {
        return
    }

    print(placemark.name)
        // 200 Queen St
    print(placemark.qualifiedName)
        // 200 Queen St, Saint John, New Brunswick E2L 2X1, Canada

    let coordinate = placemark.location.coordinate
    print("\(coordinate.latitude), \(coordinate.longitude)")
        // 45.270093, -66.050985

    #if !os(tvOS)
        let formatter = CNPostalAddressFormatter()
        print(formatter.string(from: placemark.postalAddress!))
            // 200 Queen St
            // Saint John New Brunswick E2L 2X1
            // Canada
    #endif
}
```

### Retrieve places near a location

This is often called **reverse geocoding**. Request feature data located at the input `{longitude},{latitude}` coordinates. The response includes at most one result from each type, unless the `limit` parameter was used in the request.

[Try this in the API Playground](https://www.mapbox.com/api-playground/#/reverse-geocoding/).

```endpoint
GET /geocoding/v5/{mode}/{longitude},{latitude}.json
```

URL Parameter | Description
----------|------------
`mode` | Either `mapbox.places` for ephemeral geocoding, or `mapbox.places-permanent` for storing results and batch geocoding.

Query Parameter | Description
----------|------------
`country`<br /> (optional) |  Limit results to one or more countries. Options are [ISO 3166 alpha 2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country codes separated by commas.
`types`<br /> (optional) |  Filter results by one or more type. Options are: `country`, `region`, `postcode`, `district`, `place`, `locality`, `neighborhood`, `address`, `poi`, and `poi.landmark`. Multiple options can be comma-separated. Note that `poi.landmark` returns a subset of the results returned by `poi`.
`limit`<br /> (optional) |  Limit the maximum number of results. The default is `1`, and the maximum is `5`. If using this option, a single `types` is required.
`language`<br /> (optional) | Specify the language to use for response text. Options are [IETF language tags](https://en.wikipedia.org/wiki/IETF_language_tag) comprised of a mandatory [ISO 639-1 language code](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) and optionally one or more IETF subtags for country or script. More than one value can also be specified, separated by commas.
`reverseMode`<br /> (optional) | Set the factors that are used to sort nearby results. Options are `distance` (default) and `score`.

Note that a request that includes a query for coordinates outside of the specified `country` will not throw an HTTP error, but the response will have an empty `features` array.

The optional `reverseMode` parameter controls whether feature prominence should be considered when returning features for a reverse geocoding query. By default, a reverse geocode returns the nearest feature. When you set `reverseMode=score`, the notability of features within approximately 1 kilometer of the queried point will be considered along with proximity. This option is not available when the `limit` parameter is set to a value greater than 1.

#### Example request

```curl
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/-73.989,40.733.json?access_token={your_access_token}"

# filter results to only include points of interest
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/-73.989,40.733.json?types=poi&access_token={your_access_token}"
```

```javascript
geocodingClient
  .reverseGeocode({
    query: [-95.4431142, 33.6875431],
    limit: 2
  })
  .send()
  .then(response => {
      // match is a GeoJSON document with geocoding matches
    const match = response.body;
  });
```

```python
response = geocoder.reverse(lon=-73.989, lat=40.733)
features = sorted(response.geojson()['features'], key=lambda x: x['place_name'])
for f in features:
    print('{place_name}: {id}'.format(**f))
# 120 East 13th Street, Manhattan, New York, New York 10003... address...
# Greenwich Village... neighborhood...
# Manhattan... locality...
# New York, New York... postcode...
# New York, New York... place...
# New York... region...
# United States: country...

# Limit results to a specific location type
response = geocoder.reverse(lon=-73.989, lat=40.733, limit=1, types=['country'])
features = response.geojson()['features']
len(features)
# 1
print('{place_name}: {id}'.format(**features[0]))
# United States: country...

# Filter by country code
response = geocoder.forward("200 queen street", country=['us'])
any(['Canada' in f['place_name'] for f in response.geojson()['features']])
# False

# Filter results to only include points of interest
response = geocoder.reverse(
    lon=-73.989, lat=40.733, types=['poi'])
features = response.geojson()['features']
all([f['id'].startswith('poi') for f in features])
# True
```

```bash
mapbox geocode --reverse '[-77.4371, 37.5227]'
```

```java
// the location where you'd like to get info about
Point searchLocationPoint = Point.fromLngLat(2.2869, 48.8590);

MapboxGeocoding client = MapboxGeocoding.builder()
  .accessToken("{your_access_token}")
  .query(searchLocationPoint)
  .geocodingTypes(GeocodingCriteria.TYPE_POI)
  .mode(GeocodingCriteria.MODE_PLACES)
  .build();
```

```objc
MBReverseGeocodeOptions *options = [[MBReverseGeocodeOptions alloc] initWithCoordinate: CLLocationCoordinate2DMake(40.733, -73.989)];
// Or perhaps: [[MBReverseGeocodeOptions alloc] initWithLocation:locationManager.location]

NSURLSessionDataTask *task = [geocoder geocodeWithOptions:options completionHandler:^(NSArray<MBGeocodedPlacemark *> * _Nullable placemarks, NSString * _Nullable attribution, NSError * _Nullable error) {
    MBPlacemark *placemark = placemarks[0];
    NSLog(@"%@", placemark.imageName);
        // telephone
    NSLog(@"%@", [placemark.genres componentsJoinedByString:@", "]);
        // computer, electronic
    NSLog(@"%@", placemark.administrativeRegion.name);
        // New York
    NSLog(@"%@", placemark.administrativeRegion.code);
        // US-NY
    NSLog(@"%@", placemark.place.wikidataItemIdentifier);
        // Q60
}];
```

```swift
let options = ReverseGeocodeOptions(coordinate: CLLocationCoordinate2D(latitude: 40.733, longitude: -73.989))
// Or perhaps: ReverseGeocodeOptions(location: locationManager.location)

let task = geocoder.geocode(options) { (placemarks, attribution, error) in
    guard let placemark = placemarks?.first else {
        return
    }

    print(placemark.imageName ?? "")
        // telephone
    print(placemark.genres?.joined(separator: ", ") ?? "")
        // computer, electronic
    print(placemark.administrativeRegion?.name ?? "")
        // New York
    print(placemark.administrativeRegion?.code ?? "")
        // US-NY
    print(placemark.place?.wikidataItemIdentifier ?? "")
        // Q60
}
```

### Batch requests

_This feature is only available with the `mapbox.places-permanent` mode._

```curl
curl "https://api.mapbox.com/geocoding/v5/mapbox.places-permanent/20001;20009;22209.json?access_token={your_access_token}"
```

Batch requests have the same parameters as normal requests, but can include
more than one query by separating queries with the `;` character. Each
query should be URL encoded, but the `;` character should not be encoded and
should be included verbatim.

With the `mapbox.places-permanent` mode, you can make up to 50 forward
or reverse geocoding queries in a single request. The response is
an array of individual geocoder responses.
Each query in a batch request counts individually against
your account's rate limits.

```swift
let options = ForwardBatchGeocodeOptions(queries: ["skyline chili", "gold star chili"])
options.focalLocation = locationManager.location
options.allowedScopes = .pointOfInterest

let task = geocoder.batchGeocode(options) { (placemarksByQuery, attributionsByQuery, error) in
    guard let placemarksByQuery = placemarksByQuery else {
        return
    }

    let nearestSkyline = placemarksByQuery[0][0].location
    let distanceToSkyline = nearestSkyline.distance(from: locationManager.location)
    let nearestGoldStar = placemarksByQuery[1][0].location
    let distanceToGoldStar = nearestGoldStar.distance(from: locationManager.location)

    let distance = LengthFormatter().string(fromMeters: min(distanceToSkyline, distanceToGoldStar))
    print("Found a chili parlor \(distance) away.")
}
```

### POI categories

<!-- preview -->

POI category search supports forward geocoding requests of `poi` feature types in a queried category. Using the `proximity` query parameter with POI category search returns points of interest local to a provided location; for example, restaurants near a user. Any category that is returned in the `properties.category` property of the response object is supported.

Here are some of the most common POI categories:

Category | | | |
--- | --- | --- | --- | ---
bakery | bank | bar | cafe | church
cinema | coffee | concert | fast food | finance
gallery | historic | hotel | landmark | museum
music | park | pizza | restaurant | retail
school | shop | tea | theater | university

The current list of POI categories is subject to change. A full list of supported categories will be made available in the future.

```endpoint
GET /geocoding/v5/{mode}/{category}.json
```

#### Example request

```curl
$ curl "https://api.mapbox.com/geocoding/v5/mapbox.places/coffee.json?&proximity=-77.032,38.912&limit=10&types=poi&access_token={your_access_token}"
```
