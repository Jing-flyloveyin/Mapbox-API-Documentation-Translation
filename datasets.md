## Datasets

A dataset is an editable collection of [GeoJSON features](https://tools.ietf.org/html/rfc7946). The Datasets API offers persistent storage for custom geographic data. It supports reading, creating, updating, and removing features from a dataset. To serve your geographic data at scale, convert your dataset into a tileset using [the Uploads API](https://www.mapbox.com/api-documentation/#create-an-upload).

Using the Datasets API involves interacting with two types of resources: **datasets** and **features**. Datasets contain one or more collections of features.

**Restrictions and limits**

- The Dataset API is limited on a per dataset basis.
- The default rate limit is 40 writes and 480 reads per minute.
- Dataset names are limited to 60 characters, and dataset descriptions are limited to 300 characters.
- Each feature cannot exceed 1023 KB compressed. Features are compressed server-side using [geobuf](https://github.com/mapbox/geobuf).

If you require a higher rate limit, [contact us](https://www.mapbox.com/contact/).

```python
from mapbox import Datasets
```

```javascript
const mbxDatasets = require('@mapbox/mapbox-sdk/services/datasets');
const datasetsClient = mbxDatasets({ accessToken: '{your_access_token}' });
```

### The dataset object

The **dataset** object contains information pertinent to a specific dataset. Each dataset object contains the following properties:

Property | Description
--- | ---
`owner` | The username of the dataset owner.
`id` | The ID for an existing dataset.
`created` | A timestamp indicating when the dataset was created.
`modified` | A timestamp indicating when the dataset was last modified.
`bounds` | The extent of features in the dataset in the format [`west`, `south`, `east`, `north`].
`features` | The number of features in the dataset.
`size` | The size of the dataset in bytes.
`name`<br /> (optional) | The name of the dataset.
`description`<br /> (optional) | A description of the dataset.

#### The dataset object

```json
{
  "owner": "{username}",
  "id": "{dataset_id}",
  "created": "{timestamp}",
  "modified": "{timestamp}",
  "bounds": [-10, -10, 10, 10],
  "features": 100,
  "size": 409600,
  "name": "{name}",
  "description": "{description}"
}
```

### List datasets

```endpoint
GET /datasets/v1/{username} datasets:list
```

List all the datasets that belong to a particular account.

URL parameters | Description
--- | ---
`username` | The username of the account for which to list datasets.

#### Example request

```curl
$ curl "https://api.mapbox.com/datasets/v1/{username}?access_token={your_access_token}"
```

```bash
$ mapbox datasets list
```

```javascript
datasetsClient
  .listDatasets()
  .send()
  .then(response => {
    const datasets = response.body;
  });

datasetsClient
  .listDatasets()
  .eachPage((error, response, next) => {
    // Handle error or response and call next.
  });
```

```python
datasets = Datasets()
datasets.list()
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
    "owner": "{username}",
    "id": "{dataset_id}",
    "created": "{timestamp}",
    "modified": "{timestamp}",
    "bounds": [-10, -10, 10, 10],
    "features": 100,
    "size": 409600,
    "name": "{name}",
    "description": "{description}"
  },
  {
    "owner": "{username}",
    "id": "{dataset_id}",
    "created": "{timestamp}",
    "modified": "{timestamp}",
    "bounds": [-10, -10, 10, 10],
    "features": 100,
    "size": 409600,
    "name": "{name}",
    "description": "{description}"
  }
]
```

### Create dataset

```endpoint
POST /datasets/v1/{username} datasets:write
```

Create a new, empty dataset.

URL parameters | Description
--- | ---
`username` | The username of the account to which the new dataset belongs.

#### Example request

```curl
curl -X POST "https://api.mapbox.com/datasets/v1/{username}?access_token={your_access_token}" \
  -d @request.json \
  --header "Content-Type:application/json
```

```bash
mapbox datasets create
```

```javascript
datasetsClient
  .createDataset({
    name: 'example',
    description: 'An example dataset'
  })
  .send()
  .then(response => {
    const datasetMetadata = response.body;
  });
```

```python
datasets.create(name='example', description='An example dataset')
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

#### Example request body

```json
{
  "name": "foo",
  "description": "bar"
}
```

#### Example response

```json
{
  "owner": "{username}",
  "id": "{dataset_id}",
  "created": "{timestamp}",
  "modified": "{timestamp}",
  "bounds": [-10, -10, 10, 10],
  "features": 100,
  "size": 409600,
  "name": "foo",
  "description": "bar"
}
```

### Retrieve a dataset

```endpoint
GET /datasets/v1/{username}/{dataset_id} datasets:read
```

Retrieve information about a single existing dataset.

URL parameters | Description
--- | ---
`username` | The username of the account to which the dataset belongs.
`dataset_id` | The ID of the dataset to be retrieved.

#### Example request

```curl
curl "https://api.mapbox.com/datasets/v1/{username}/{dataset_id}?access_token={your_access_token}"
```

```bash
mapbox dataset read-dataset dataset-id
```

```python
datasets.read_dataset(dataset_id).json()
```

```javascript
datasetsClient
  .getMetadata({
    datasetId: 'dataset-id'
  })
  .send()
  .then(response => {
    const datasetMetadata = response.body;
  })
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
  "owner": "{username}",
  "id": "{dataset_id}",
  "created": "{timestamp}",
  "modified": "{timestamp}",
  "bounds": [-10, -10, 10, 10],
  "features": 100,
  "size": 409600,
  "name": "{name}",
  "description": "{description}"
}
```

### Update a dataset

```endpoint
PATCH /datasets/v1/{username}/{dataset_id} datasets:write
```

Update the properties of a specific dataset. The request body must be valid JSON.

URL parameters | Description
--- | ---
`username` | The username of the account to which the dataset belongs.
`dataset_id` | The ID of the dataset to be updated.

#### Example request

```curl
curl -X PATCH "https://api.mapbox.com/datasets/v1/{username}/{dataset_id}?access_token={your_access_token}" \
  -d @data.json
```

```python
datasets.update_dataset(
    dataset_id,
    name='updated example',
    description='An updated example dataset'
    ).json()
```

```bash
mapbox dataset update-dataset dataset-id
```

```javascript
datasetsClient
  .updateMetadata({
    datasetId: 'dataset-id',
    name: 'foo'
  })
  .send()
  .then(response => {
    const datasetMetadata = response.body;
  });
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

#### Example request body

```json
{
  "name": "foo",
  "description": "bar"
}
```

#### Example response

```json
{
  "owner": "{username}",
  "id": "{dataset_id}",
  "created": "{timestamp}",
  "modified": "{timestamp}",
  "bounds": [-10, -10, 10, 10],
  "features": 100,
  "size": 409600,
  "name": "foo",
  "description": "bar"
}
```

### Delete a dataset

```endpoint
DELETE /datasets/v1/{username}/{dataset_id} datasets:write
```

Delete a specific dataset. All the features contained in the dataset will be deleted too.

URL parameters | Description
--- | ---
`username` | The username of the account to which the dataset belongs.
`dataset_id` | The ID of the dataset to be deleted.

#### Example request

```curl
curl -X DELETE "https://api.mapbox.com/datasets/v1/{username}/{dataset_id}?access_token={your_access_token}"
```

```bash
mapbox dataset delete-dataset dataset-id
```

```python
datasets.delete_dataset(dataset_id)
```

```javascript
datasetsClient
  .deleteDataset({
    datasetId: 'dataset-id'
  })
  .send()
  .then(response => {
    // Dataset is successfully deleted.
  });
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

> HTTP 204

### The feature object

A **feature** is a GeoJSON Feature object representing a feature in the dataset. GeometryCollections and null geometries are not supported. For a full list of GeoJSON Feature properties, see the [GeoJSON specification](https://tools.ietf.org/html/rfc7946#section-3.2).

#### The feature object

```json
{
  "id": "{feature_id}",
  "type": "Feature",
  "properties": {
    "prop0": "value0"
  },
  "geometry": {
    "coordinates": [102, 0.5],
    "type": "Point"
  }
}
```

### List features

```endpoint
GET /datasets/v1/{username}/{dataset_id}/features datasets:read
```

List all the features in a dataset. The response body will be a GeoJSON FeatureCollection.

This endpoint supports [pagination](#pagination) so that you can list many features.

URL parameters | Description
--- | ---
`username` | The username of the account to which the dataset belongs.
`dataset_id` | The ID of the dataset for which to retrieve features.

Query parameters | Description
--- | ---
`limit`<br> (optional) | The maximum number of features to return, from `1` to `100`. The default is `10`.
`start`<br> (optional) | The ID of the feature after which to start the listing. See the [pagination](#pagination) section for details.

#### Example request

```curl
curl "https://api.mapbox.com/datasets/v1/{username}/{dataset_id}/features?access_token={your_access_token}"

# Limit results to 10 features
curl "https://api.mapbox.com/datasets/v1/{username}/{dataset_id}/features?limit=10&access_token={your_access_token}"

# Use pagination to start the listing after the feature with ID f6d9
curl "https://api.mapbox.com/datasets/v1/{username}/{dataset_id}/features?start=f6d9&access_token={your_access_token}"
```

```bash
mapbox dataset list-features dataset-id
```

```python
datasets.list_features(dataset_id).json()
```

```javascript
datasetsClient
  .listFeatures({
    datasetId: 'dataset-id'
  })
  .send()
  .then(response => {
    const features = response.body;
  });
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
      "id": "{feature_id}",
      "type": "Feature",
      "properties": {
        "prop0": "value0"
      },
      "geometry": {
        "coordinates": [ 102,0.5 ],
        "type": "Point"
      }
    },
    {
      "id": "{feature_id}",
      "type": "Feature",
      "properties": {
        "prop0": "value0"
      },
      "geometry": {
        "coordinates": [
          [ 102, 0 ],
          [ 103, 1 ],
          [ 104, 0 ],
          [ 105, 1 ]
        ],
        "type": "LineString"
      }
    }
  ]
}
```

### Insert or update a feature

```endpoint
PUT /datasets/v1/{username}/{dataset_id}/features/{feature_id} datasets:write
```

Insert or update a feature in a specified dataset:

- **Insert**: If a feature with the given `feature_id` does not already exist, a new feature will be created.
- **Update**: If a feature with the given `feature_id` does exist already, it will be replaced.

If you are inserting a feature, you must add the feature as the body of the `PUT` request. This should be one individual GeoJSON feature, not a GeoJSON FeatureCollection. If the GeoJSON feature has a top-level `id` property, it must match the `feature_id` you use in the URL endpoint.

URL parameters | Description
--- | ---
`username` | The username of the account to which the dataset belongs.
`dataset_id` | The ID of the dataset for which to insert or update features.
`feature_id` | The ID of the feature to be inserted or updated.

#### Example request

```curl
curl "https://api.mapbox.com/datasets/v1/{username}/{dataset_id}/features/{feature_id}?access_token={your_access_token}" \
  -X PUT \
  -H "Content-Type: application/json" \
  -d @file.geojson
```

```bash
mapbox dataset put-feature dataset-id feature-id 'geojson-feature'
```

```javascript
datasetsClient
  .putFeature({
    datasetId: 'dataset-id',
    featureId: 'null-island',
    feature: {
      "type": "Feature",
      "properties": { "name": "Null Island" },
      "geometry": {
        "type": "Point",
        "coordinates": [0, 0]
      }
    }
  })
  .send()
  .then(response => {
    const feature = response.body;
  });
```

```python
feature = {
    'type': 'Feature', 'id': 'feature-id', 'properties': {'name': 'Insula Nulla'},
    'geometry': {'type': 'Point', 'coordinates': [0, 0]}}
datasets.update_feature('dataset-id', 'feature-id', feature)
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

#### Example request body

```json
{
  "id": "{feature_id}",
  "type": "Feature",
  "geometry": {
    "type": "Polygon",
    "coordinates": [
      [
        [ 100, 0 ],
        [ 101, 0 ],
        [ 101, 1 ],
        [ 100, 1 ],
        [ 100, 0 ]
      ]
    ]
  },
  "properties": {
    "prop0": "value0"
  }
}
```

#### Example response

```json
{
  "id": "{feature_id}",
  "type": "Feature",
  "geometry": {
    "type": "Polygon",
    "coordinates": [
      [
        [ 100, 0 ],
        [ 101, 0 ],
        [ 101, 1 ],
        [ 100, 1 ],
        [ 100, 0 ]
      ]
    ]
  },
  "properties": {
    "prop0": "value0"
  }
}
```

### Retrieve a feature

```endpoint
GET /datasets/v1/{username}/{dataset_id}/features/{feature_id} datasets:read
```

Retrieve a specific feature from a dataset.

URL parameters | Description
--- | ---
`username` | The username of the account to which the dataset belongs.
`dataset_id` | The ID of the dataset from which to retrieve a feature.
`feature_id` | The ID of the feature to be retrieved.

#### Example request

```curl
curl "https://api.mapbox.com/datasets/v1/{username}/{dataset_id}/features/{feature_id}?access_token={your_access_token}"
```

```bash
mapbox dataset read-feature dataset-id feature-id
```

```javascript
datasetsClient
  .getFeature({
    datasetId: 'dataset-id',
    featureId: 'feature-id'
  })
  .send()
  .then(response => {
    const feature = response.body;
  });
```

```python
datasets.read_feature(dataset_id, '2').json()
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
  "id": "{feature_id}",
  "type": "Feature",
  "geometry": {
    "type": "Polygon",
    "coordinates": [
      [
        [ 100, 0 ],
        [ 101, 0 ],
        [ 101, 1 ],
        [ 100, 1 ],
        [ 100, 0 ]
      ]
    ]
  },
  "properties": {
    "prop0": "value0"
  }
}
```

### Delete a feature

```endpoint
DELETE /datasets/v1/{username}/{dataset_id}/features/{feature_id} datasets:write
```

Remove a specific feature from a dataset.

URL parameters | Description
--- | ---
`username` | The username of the account to which the dataset belongs.
`dataset_id` | The ID of the dataset from which to delete a feature.
`feature_id` | The ID of the feature to be deleted.

#### Example request

```curl
curl -X DELETE "https://api.mapbox.com/datasets/v1/{username}/{dataset_id}/features/{feature_id}?access_token={your_access_token}"
```

```javascript
datasetsClient
  .deleteFeature({
    datasetId: 'dataset-id',
    featureId: 'feature-id'
  })
  .send()
  .then(response => {
    // Feature is successfully deleted.
  });
```

```python
datasets.delete_feature(dataset_id, feature_id)
```

```bash
mapbox dataset delete-feature dataset-id feature-id
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

> HTTP 204
