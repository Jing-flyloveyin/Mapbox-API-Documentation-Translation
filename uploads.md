## Uploads

Mapbox Uploads API将地理数据转换为可与地图和地理应用一起使用的瓦片。鉴于各种地理空间格式，它对投影进行标准化并生成多种缩放等级的切片以便在网络上查看数据。

上传工作流程遵循以下步骤：

1. 请求允许您暂存文件的临时S3凭据。_跳转至 [取回S3凭据部分](#retrieve-s3-credentials) 。_
1. 使用S3客户端通过这些凭据来上传文件至Mapbox的S3 staging bucket。_通过 [使用cURL教程上传至Mapbox](https://www.mapbox.com/help/upload-curl/) 了解此过程的更多信息。_
1. 使用暂存文件的URL创建上传。_跳转至 [创建上传部分](#create-an-upload) 。_
1. 在其正被处理为一个瓦片的同时检索上传的状态。_跳转至 [检索上传状态部分](#retrieve-upload-status) 。_

**注意：** 该文档讨论了如何以编程方式与Mapbox Uploads API进行交互。有关如何使用Uploads API将文件转存至Amazon S3并创建上载的分步指南，使用 [使用cURL教程上传至Mapbox](https://www.mapbox.com/help/upload-curl/)。

**限制**

- 瓦片文件名限制为64个字符。
- Uploads API支持各种文件类型的不同文件大小：

文件类型 | 大小限制
--- | ---
TIFF 和 GeoTIFF | 10 GB
MBTiles | 25 GB
GeoJSON | 1 GB
CSV | 1 GB
KML | 260 MB
GPX | 260 MB
Shapefile（压缩） | 260 MB
Mapbox Dataset | 1 GB

**错误信息**

查看可能的上传错误列表，访问 [上传错误页面](https://www.mapbox.com/help/uploads/#errors) 。

```python
from mapbox import Uploader
```

```javascript
const mbxUploads = require('@mapbox/mapbox-sdk/services/uploads');
const uploadsClient = mbxUploads({ accessToken: '{your_access_token}' });
```

### Retrieve S3 credentials

```endpoint
POST /uploads/v1/{username}/credentials uploads:write
```

Mapbox提供了一个Amazon S3 bucket，用于在处理上传时暂存文件。此端点允许您检索临时S3凭据以在暂存过程中使用。

**注意：** 在您可以在Mapbox提供的Amazon S3 bucket中暂存文件之前，必须执行此步骤。所有上传必须在此Amazon S3 bucket中暂存，然后才能上传到您的Mapbox帐户。要了解有关如何上传到Amazon S3的更多信息，查阅 [使用cURL教程上传至Mapbox](https://www.mapbox.com/help/upload-curl/#stage-your-file-on-amazon-s3) 。

URL参数 | 描述
--- | ---
`username` | 要上传瓦片的帐户的用户名。
#### 请求示例

```curl
curl -X POST "https://api.mapbox.com/uploads/v1/{username}/credentials?access_token={your_access_token}"
```

```javascript
const AWS = require('aws-sdk');

const getCredentials = () => {
  return uploadsClient
    .createUploadCredentials()
    .send()
    .then(response => response.body);
}

const putFileOnS3 = (credentials) => {
  const s3 = new AWS.S3({
    accessKeyId: credentials.accessKeyId,
    secretAccessKey: credentials.secretAccessKey,
    sessionToken: credentials.sessionToken,
    region: 'us-east-1'
  });
  return s3.putObject({
    Bucket: credentials.bucket,
    Key: credentials.key,
    Body: fs.createReadStream('/path/to/file.mbtiles')
  }).promise();
};

getCredentials().then(putFileOnS3);
```

```python
service = Uploader()
with open('data.geojson', 'r') as src:
    # Acquisition of credentials, staging of data, and upload
    # finalization is done by a single method in the Python SDK.
    upload_resp = service.upload(src, '{username}.data')
```

```bash
# 无法通过Mapbox CLI访问AWS
```

```java
// 无法使用Mapbox Java SDK访问此API
```

```objc
// 无法使用Mapbox Objective-C库访问此API
```

```swift
// 无法使用Mapbox Swift库访问此API
```

#### 响应示例

```json
{
  "accessKeyId": "{accessKeyId}",
  "bucket": "somebucket",
  "key": "hij456",
  "secretAccessKey": "{secretAccessKey}",
  "sessionToken": "{sessionToken}",
  "url": "{url}"
}
```

**响应主体**

响应主体是一个包含以下属性的JSON对象：

属性 | 描述
---- | -----------
`accessKeyId` | AWS访问密钥ID
`bucket` | S3 bucket名称
`key` | 要暂存数据的唯一键
`secretAccessKey` | AWS秘密访问密钥
`sessionToken` | 临时安全令牌
`url` | 文件的目标URL

使用这些凭据通过 [AWS CLI](http://docs.aws.amazon.com/cli/latest/reference/s3/index.html) 将提供的密钥存储在提供的存储桶中或您选择AWS SDK。存储桶位于AWS区域 `us-east-1`.

**AWS CLI使用示例**

```
$ export AWS_ACCESS_KEY_ID={accessKeyId}
$ export AWS_SECRET_ACCESS_KEY={secretAccessKey}
$ export AWS_SESSION_TOKEN={sessionToken}
$ aws s3 cp /path/to/file s3://{bucket}/{key} --region us-east-1
```

### Create an upload

```endpoint
POST /uploads/v1/{username} uploads:write
```

使用临时S3凭据将文件传输到Mapbox的临时存储桶后，可以使用文件的URL和目标瓦片ID触发瓦片的生成。

上传的文件_必须_位于Mapbox提供的存储桶中。来自其他S3存储桶或URL的资源请求将失败。

URL参数 | 描述
--- | ---
`username` | 您要上传的帐户的用户名

**请求主体**

请求主体必须是包含以下属性的JSON对象：

属性 | 描述
---- | ------
`tileset` | 要创建或替换的地图ID，格式`username.nameoftileset`。限制为32个字符。此字符限制不包括用户名。 唯一允许的特殊字符是`-`和`_`。
`url` | 凭据请求中提供的S3对象的HTTPS URL，或要上载的现有Mapbox数据集的[数据集ID](https://www.mapbox.com/help/define-dataset-id/)。
`name`<br /> （可选） | 瓦片文件名。限制为64个字符。

如果重复使用`tileset`值，此操作将替换现有数据。使用随机值确保创建新的瓦片，或首先检查现有的瓦片。

#### 请求示例

```curl
curl -X POST -H "Content-Type: application/json" -H "Cache-Control: no-cache" -d '{
  "url": "http://{bucket}.s3.amazonaws.com/{key}",
  "tileset": "{username}.{tileset-name}"
}' 'https://api.mapbox.com/uploads/v1/{username}?access_token=secret_access_token'
```

```javascript
// 调用createUploadCredentials的响应
const credentials = {
  accessKeyId: '{accessKeyId}',
  bucket: '{bucket}',
  key: '{key}',
  secretAccessKey: '{secretAccessKey}',
  sessionToken: '{sessionToken}',
  url: '{s3 url}'
};

uploadsClient
  .createUpload({
    mapId: `${myUsername}.${myTileset}`,
    url: credentials.url
  })
  .send()
  .then(response => {
    const upload = response.body;
  });
```

```python
with open('data.geojson', 'r') as src:
    # 获取凭证，暂存数据和上传
    # 最终化由Python SDK中的单个方法完成。
    upload_resp = service.upload(src, '{username}.data')
```

```bash
mapbox upload username.data data.geojson
```

```java
// 无法使用Mapbox Java SDK访问此API
```

```objc
// 无法使用Mapbox Objective-C库访问此API
```

```swift
// 无法使用Mapbox Swift库访问此API
```

#### 上传Mapbox数据集的请求示例正文（不需要AWS S3存储桶）

```json
{
  "tileset": "{username}.mytileset",
  "url": "mapbox://datasets/{username}/{dataset}",
  "name": "example-dataset"
}
```

#### 响应示例

```json
{
  "complete": false,
  "tileset": "example.markers",
  "error": null,
  "id": "hij456",
  "name": "example-markers",
  "modified": "{timestamp}",
  "created": "{timestamp}",
  "owner": "{username}",
  "progress": 0
}
```

**Response body**

The response body is a JSON object that contains the following properties:

Property | Description
---- | -----------
`complete` | Whether the upload is complete (`true`) or not complete (`false`).
`tileset` | The ID of the tileset that will be created or replaced if upload is successful.
`error` | If `null`, the upload is in progress or has successfully completed. Otherwise, provides a brief explanation of the error.
`id` | The unique identifier for the upload.
`name` | The name of the upload.
`modified` | A timestamp indicating when the upload resource was last modified.
`created` | A timestamp indicating when the upload resource was created.
`owner` | The unique identifier for the owner's account.
`progress` | The progress of the upload, expressed as a float between `0` (started) and `1` (completed).

### Retrieve upload status

```endpoint
GET /uploads/v1/{username}/{upload_id} uploads:read
```

Upload processing is fast but not immediate. Once an upload is created, you can track its status. Uploads have a `progress` property that start at `0` and end at `1` when an upload is complete. If there's an error processing an upload, the `error` property will include an [error message](https://www.mapbox.com/help/uploads/#errors).

URL parameters | Description
--- | ---
`username` | The username of the account for which you are requesting an upload status.
`upload_id` | The ID of the upload. Provided in the response body to a successful upload request.

**Response body**

The response body is a JSON object containing the following properties:

Property | Description
---- | -----------
`complete` | Boolean. Indicates whether the upload is complete (`true`) or not complete (`false`).
`tileset` | The ID of the tileset that will be created or replaced if upload is successful.
`error` | If `null`, the upload is in progress or has successfully completed. Otherwise, provides a brief explanation of the error.
`id` | The unique identifier for the upload.
`name` | The name of the upload.
`modified` | The timestamp for when the upload resource was last modified.
`created` | The timestamp for when the upload resource was created.
`owner` | The unique identifier for the owner's account.
`progress` | The progress of the upload, expressed as a float between `0` (started) and `1` (completed).

#### Example request

```curl
curl "https://api.mapbox.com/uploads/v1/{username}/{upload_id}?access_token={your_access_token}"
```

```python
service.status(upload_id).json()
```

```javascript
uploadsClient
  .getUpload({
    uploadId: '{upload_id}'
  })
  .send()
  .then(response => {
    const status = response.body;
  });
```

```bash
# This method cannot be accessed with Mapbox CLI
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
  "complete": true,
  "tileset": "example.markers",
  "error": null,
  "id": "hij456",
  "name": "example-markers",
  "modified": "{timestamp}",
  "created": "{timestamp}",
  "owner": "{username}",
  "progress": 1
}
```

### Retrieve recent upload statuses

```endpoint
GET /uploads/v1/{username} uploads:list
```

Retrieve multiple upload statuses at the same time, sorted by the most recently created. This request returns the same information as an individual upload status request does, but for all an account's recent uploads. The list is limited to 1MB of JSON.

This endpoint supports [pagination](#pagination) so that you can list many uploads.

URL parameters | Description
--- | ---
`username` | The username of the account for which you are requesting upload statuses.

The results from this endpoint can be further modified with the following optional parameters:

Query parameter | Description
--- | ---
`reverse`<br>(optional) | Set to `true` to reverse the default sort order of the results listing.
`limit`<br>(optional) | The maximum number of statuses to return, up to `100`.

#### Example request

```curl
curl "https://api.mapbox.com/uploads/v1/{username}?access_token={your_access_token}"

# Return at most 10 upload statuses, in chronological order
curl "https://api.mapbox.com/uploads/v1/{username}?reverse=true&limit=10&access_token={your_access_token}"
```

```python
service.list().json()
```

```javascript
uploadsClient
  .listUploads()
  .send()
  .then(response => {
    const uploads = response.body;
  });
```

```bash
This method cannot be accessed with Mapbox CLI
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
[{
  "complete": true,
  "tileset": "example.mbtiles",
  "error": null,
  "id": "abc123",
  "name": null,
  "modified": "2014-11-21T19:41:10.000Z",
  "created": "2014-11-21T19:41:10.000Z",
  "owner": "example",
  "progress": 1
}, {
  "complete": false,
  "tileset": "example.foo",
  "error": null,
  "id": "xyz789",
  "name": "foo",
  "modified": "2014-11-21T19:41:10.000Z",
  "created": "2014-11-21T19:41:10.000Z",
  "owner": "example",
  "progress": 0
}]
```

### Remove an upload status

```endpoint
DELETE /uploads/v1/{username}/{upload_id} uploads:write
```

Remove the status of a completed upload from the upload listing.

Uploads are only statuses, so removing an upload from the listing doesn't delete the associated tileset. Tilesets can only be deleted from within Mapbox Studio. An upload status cannot be removed from the upload listing until after it has completed.

URL parameters | Description
--- | ---
`username` | The username of the associated account.
`upload_id` | The ID of the upload. Provided in the response body to a successful upload request.

#### Example request

```curl
$ curl -X DELETE "https://api.mapbox.com/uploads/v1/{username}/{upload_id}?access_token={your_access_token}"
```

```javascript
uploadsClient
  .deleteUpload({
    uploadId: '{upload_id}'
  })
  .send()
  .then(response => {
    // Upload successfully deleted.
  });
```

```python
service.delete("{upload_id}")
# <Response [204]>
```

```bash
# This method cannot be accessed with Mapbox CLI
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
