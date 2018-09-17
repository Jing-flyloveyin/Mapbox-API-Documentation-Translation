## 数据集

数据集（Dataset）是一个包含许多[GeoJSON features](https://tools.ietf.org/html/rfc7946)的数组。数据集API提供了自定义地理数据的持久存储。 它支持从数据集中读取，创建，更新和删除功能。 如想要更好的大规模投放地理数据在不同比例尺，请使用[上传API](https://www.mapbox.com/api-documentation/#create-an-upload)将数据集转换为切片集。

使用数据集API涉及与两种类型的资源交互：**数据集**和**要素**。数据集包含一个或多个要素集合。

**已知约束和限制**

- 数据集API基于每个数据集限制。
- 默认速率限制为每分钟40次写入和480次读取。
- 数据集名称限制为60个字符，数据集描述限制为300个字符。
- 每个功能压缩后不能超过1023KB。使用[geobuf](https://github.com/mapbox/geobuf)在服务器端压缩功能。

如果您需要更高的费率限制，[请与我们联络](https://www.mapbox.com/contact/)。

```python
from mapbox import Datasets
```

```javascript
const mbxDatasets = require('@mapbox/mapbox-sdk/services/datasets');
const datasetsClient = mbxDatasets({ accessToken: '{your_access_token}' });
```

### 数据集对象

**数据集**对象包含与特定数据集相关的信息。每个数据集对象包含以下属性：

属性 | 描述
--- | ---
`owner` | 数据集所有者的用户名。
`id` | 现有数据集的ID。
`created` | 指示何时创建数据集的时间戳。
`modified` | 指示上次修改数据集的时间戳。
`bounds` | 格式中数据集中要素的范围[`west`, `south`, `east`, `north`]。
`features` | 数据集中的要素数。
`size` | 数据集的大小（以字节为单位）。
`name`<br /> (可选) | 数据集的名称。
`description`<br /> (可选) | 数据集的描述。

#### 数据集对象

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

### 数据集对象

```endpoint
GET /datasets/v1/{username} datasets:list
```

列出属于特定帐户的所有数据集。


URL 参数| 描述
--- | ---
`username` | 新数据集所属的帐户的用户名。

#### 示例请求

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
// 使用Mapbox Java SDK无法访问此API
```

```objc
// 无法使用Mapbox Objective-C库访问此API
```

```swift
// 使用Mapbox Swift库无法访问此API
```

#### 示例响应

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

### 创建数据集

```endpoint
POST /datasets/v1/{username} datasets:write
```

创建一个新的空数据集。

URL 参数| 描述
--- | ---
`username` | 新数据集所属的帐户的用户名。

#### 示例请求

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
// 使用Mapbox Java SDK无法访问此API
```

```objc
// 无法使用Mapbox Objective-C库访问此API
```

```swift
// 使用Mapbox Swift库无法访问此API
```

#### 示例请求主体

```json
{
  "name": "foo",
  "description": "bar"
}
```

#### 示例响应

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

### 检索数据集

```endpoint
GET /datasets/v1/{username}/{dataset_id} datasets:read
```
检索有关单个现有数据集的信息。


URL 参数| 描述
--- | ---
`username` | 数据集所属帐户的用户名。
`dataset_id` | 要检索的数据集的ID。

#### 示例请求

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
// 使用Mapbox Java SDK无法访问此API
```

```objc
// 无法使用Mapbox Objective-C库访问此API
```

```swift
// 使用Mapbox Swift库无法访问此API
```

#### 示例响应

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

### 更新数据集

```endpoint
PATCH /datasets/v1/{username}/{dataset_id} datasets:write
```

更新特定数据集的属性。请求正文必须是有效的JSON。

URL 参数| 描述
--- | ---
`username` | 数据集所属帐户的用户名。
`dataset_id` | 要更新的数据集的ID。

#### 示例请求

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
// 使用Mapbox Java SDK无法访问此API
```

```objc
// 无法使用Mapbox Objective-C库访问此API
```

```swift
// 使用Mapbox Swift库无法访问此API
```

#### 示例请求主体

```json
{
  "name": "foo",
  "description": "bar"
}
```

#### 示例响应

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

### 删除数据集

```endpoint
DELETE /datasets/v1/{username}/{dataset_id} datasets:write
```

删除特定数据集。数据集中包含的所有功能也将被删除。

URL 参数| 描述
--- | ---
`username` | 数据集所属帐户的用户名。
`dataset_id` | 要更新的数据集的ID。

#### 示例请求

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
    // 数据集成功删除
  });
```

```java
// 使用Mapbox Java SDK无法访问此API
```

```objc
// 无法使用Mapbox Objective-C库访问此API
```

```swift
// 使用Mapbox Swift库无法访问此API
```

#### 示例响应

> HTTP 204

### 要素对象

**要素**是表示数据集中的特征的GeoJSON Features对象。不支持几何集和空的几何。有关GeoJSON Features属性的完整列表，请参阅[GeoJSON规范](https://tools.ietf.org/html/rfc7946#section-3.2)。

#### 要素对象

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

### 列出要素

```endpoint
GET /datasets/v1/{username}/{dataset_id}/features datasets:read
```

列出数据集中的所有功能。响应主体将是GeoJSON FeaturesCollection。

此端点支持[分页](https://www.mapbox.com/api-documentation/#pagination)，以便您可以列出许多要素。

URL 参数 | 描述
--- | ---
`username` | 数据集所属帐户的用户名。
`dataset_id` | 要检索功能的数据集的ID。

查询参数 | 描述
--- | ---
`limit`<br> (可选) | 要返回的最大要素数，从“1”到“100”。默认值为“10”。
`start`<br> (可选) | 要在其后开始列表的功能的ID。有关详细信息，请参阅[pagination]（#pagination）部分。

#### 示例请求

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
// 使用Mapbox Java SDK无法访问此API
```

```objc
// 无法使用Mapbox Objective-C库访问此API
```

```swift
// 使用Mapbox Swift库无法访问此API
```

#### 示例响应

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

### 插入或更新功能

```endpoint
PUT /datasets/v1/{username}/{dataset_id}/features/{feature_id} datasets:write
```

插入或更新指定数据集中的要素:

-  **插入**：如果尚未存在具有给定`feature_id`的要素，则将创建新要素。
-  **更新**：如果已经存在具有给定`feature_id`的特征，则将替换它。

如果要插入要素，则必须将该要素添加“PUT”作为请求的主体。这应该是一个单独的GeoJSON功能，而不是GeoJSON FeaturesCollection。如果GeoJSON功能具有顶置级别的`id`属性，则它必须与您在URL端点中使用的`feature_id`相匹配。


URL 参数 | 描述
--- | ---
`username` | 数据集所属帐户的用户名。
`dataset_id` | 要插入或更新要素的数据集的ID。
`feature_id` | 要插入或更新的功能的ID。

#### 示例请求

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
// 使用Mapbox Java SDK无法访问此API
```

```objc
// 无法使用Mapbox Objective-C库访问此API
```

```swift
// 使用Mapbox Swift库无法访问此API
```

#### 示例请求主体

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

#### 示例响应

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

### 检索功能

```endpoint
GET /datasets/v1/{username}/{dataset_id}/features/{feature_id} datasets:read
```

从数据集中检索特定要素。

URL 参数 | 描述
--- | ---
`username` | 数据集所属帐户的用户名。
`dataset_id` | 要插入或更新要素的数据集的ID。
`feature_id` | 要插入或更新的功能的ID。

#### 示例请求

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
// 使用Mapbox Java SDK无法访问此API
```

```objc
// 无法使用Mapbox Objective-C库访问此API
```

```swift
// 使用Mapbox Swift库无法访问此API
```

#### 示例响应

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

### 删除一个要素

```endpoint
DELETE /datasets/v1/{username}/{dataset_id}/features/{feature_id} datasets:write
```

从数据集中删除特定要素。

URL 参数 | 描述
--- | ---
`username` | 数据集所属帐户的用户名。
`dataset_id` | 要插入或更新要素的数据集的ID。
`feature_id` | 要插入或更新的功能的ID。

#### 示例请求

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
// 使用Mapbox Java SDK无法访问此API
```

```objc
// 无法使用Mapbox Objective-C库访问此API
```

```swift
// 使用Mapbox Swift库无法访问此API
```

#### 示例响应

> HTTP 204
