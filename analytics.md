## Analytics

<!-- preview -->

*This API is available for [Commercial and Enterprise](https://www.mapbox.com/pricing/) plans.*

The Mapbox Analytics API returns API usage for services by resource. For example, it can calculate the number of geocoding requests made in a week with a specific access token.

```python
from mapbox import Analytics
```

### Retrieve analytics

Returns the request counts per day for given resource and period.

* If the `{resourceType}` is `tokens`, the `{id}` is the complete token.
* If the `{resourceType}` is `styles`, the `{id}` is the Style ID. Not to be confused with the Style _URL_, the id is the alphanumeric segment at the end of the style: the Style URL `mapbox://styles/user/cimdoca6f00` contains the Style ID `cimdoca6f00`, so the analytics request would have the path `/analytics/v1/styles/user/cimdoca6f00`.
* If the `{resourceType}` is `tilesets`, the `{id}` is a Map ID.

```endpoint
GET /analytics/v1/{resourceType}/{username}/{id}?period={period} analytics:read
```

URL Parameter | Description
--- | ---
`resourceType` | The resource being requested. Valid resource types are `accounts`, `tokens`, `tilesets` or `styles`.
`username` | The username for the account that owns the resource.
`id` <br /> (optional) | The id for the resource. In the case of `accounts` this isn't required.
`period` <br /> (optional) | 2 comma separated dates as [ISO formatted strings](#dates). The first date must be earlier than the second. The period is inclusive of dates provided. Defaults to last 90 days if not provided. The maximum period is 1 year. If the provided dates are more than 1 year apart, an error will be returned.


#### Example Request

```curl
curl "https://api.mapbox.com/analytics/v1/tilesets/mapbox/mapbox.streets?period=2016-03-22T00:00:00.000Z,2016-03-24T00:00:00.000Z&access_token={your_access_token}"
```

```bash
# This API cannot be accessed with Mapbox CLI
```

```javascript
// This API cannot be accessed with the JavaScript SDK
```

```python
analytics = Analytics()
response = analytics.analytics('accounts', '{username}')
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
    "period": ["2016-03-22T00:00:00.000Z", "2016-03-24T00:00:00.000Z"],
    "timestamps": ["2016-03-22T00:00:00.000Z", "2016-03-23T00:00:00.000Z", "2016-03-24T00:00:00.000Z"],
    "services": {
        "mapview": [ 25, 22, 37 ],
        "static": [ 23, 20, 34 ],
        "tiles": [ 30, 39, 53 ]
    }
}
```

Responses include arrays of request counts per service.

Property | Description
--- | ---
`timestamps` | an array of dates as ISO formatted strings for each day included in the response.
`period` | a 2 element array with start and end dates as ISO formatted strings for the response period.
`services` | an object with a key per service the value of which is an array of request counts per day in the same sequence as `timestamps`.

Only services applicable to the given resource are returned in the response.

Resource type | Services returned in response
--- | ---
`accounts` | `mapview` `static` `tile` `directions` `geocode`
`tokens` | `mapview` `static` `tile` `directions` `geocode`
`tilesets` | `mapview` `static` `tile`
`styles` | `mapview` `static` `tile`

For the `styles` resource type, Static & Tile services refer to the [Static API](#static). For `tilesets`, Static & Tile services refer to the [Static (Classic)](#static-classic) and [Maps → Tiles](#retrieve-tiles) APIs.
