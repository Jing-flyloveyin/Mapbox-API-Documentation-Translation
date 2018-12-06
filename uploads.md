## Uploads 上传

Mapbox Uploads API 将地理数据转换为可用于地图和地理应用的 tilesets。它支持对多种格式的空间数据进行投影标准化，并生成多个缩放级别的切片，以便在网页端可视化。

上传流程如下：

1. 请求临时 S3 凭证用于暂存文件。_跳转到 [Retrieve S3 credentials section](#retrieve-s3-credentials)。_
1. 用这些凭证在 S3 客户端上传文件到 Mapbox 的 S3 临时存储桶。_访问 [Upload to Mapbox using cURL tutorial](https://www.mapbox.com/help/upload-curl/) 了解更多。_
1. 使用暂存文件的 URL 创建上传。_跳转到 [Create an upload section](#create-an-upload)。_
1. 在转换为 tileset 时检索上传状态。_跳转到 [Retrieve upload status section](#retrieve-upload-status)。_

**注意:** 本文档讨论如何在编程中使用 Mapbox Uploads API。有关如何使用 Uploads API 将文件暂存到 Amazon S3 并创建上传的指南，请访问 [Upload to Mapbox using cURL tutorial](https://www.mapbox.com/help/upload-curl/)。

**约束和限制**

- Tileset 名称限制为64个字符。
- Uploads API 各种文件类型的大小限制：

文件类型 | 大小限制
--- | ---
TIFF and GeoTIFF | 10 GB
MBTiles | 25 GB
GeoJSON | 1 GB
CSV | 1 GB
KML | 260 MB
GPX | 260 MB
Shapefile (zipped) | 260 MB
Mapbox Dataset | 1 GB

**错误消息**

访问[Uploads errors page](https://www.mapbox.com/help/uploads/#errors)查看可能出现的错误。

```python
from mapbox import Uploader
```

```javascript
const mbxUploads = require('@mapbox/mapbox-sdk/services/uploads');
const uploadsClient = mbxUploads({ accessToken: '{your_access_token}' });
```

### 检索 S3 凭证

```endpoint
POST /uploads/v1/{username}/credentials uploads:write
```

Mapbox 提供了一个 Amazon S3 存储桶，用于在上传时暂存文件。这允许你检索暂存过程中使用的临时 S3 凭证。

**注意:** 在 Mapbox 提供的 Amazon S3 存储桶中暂存文件之前，必须执行此步骤。所有上传必须在 Amazon S3 存储桶中暂存，然后才能上传到你的 Mapbox 帐户。了解关于如何上传至 Amazon S3 的更多信息，请阅读 [Upload to Mapbox using cURL tutorial](https://www.mapbox.com/help/upload-curl/#stage-your-file-on-amazon-s3)。

URL 参数 | 描述
--- | ---
`username` | tileset上传帐户的用户名。

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
# AWS 不支持 Mapbox CLI
```

```java
// 此 API 不支持 Mapbox Java SDK
```

```objc
// 此 API 不支持 Mapbox Objective-C 库
```

```swift
// 此 API 不支持 Mapbox Swift 库
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

**响应体**

响应体是一个包含以下属性的JSON对象：

属性 | 描述
---- | -----------
`accessKeyId` | AWS 密钥 ID
`bucket` | S3 存储桶名称
`key` | 暂存数据的唯一键
`secretAccessKey` | AWS 私钥
`sessionToken` | 临时安全令牌
`url` | 文件的目标URL

用这些凭证将数据存储至存储桶中，存储桶的密钥可用 [AWS CLI](http://docs.aws.amazon.com/cli/latest/reference/s3/index.html) 或 AWS SDK 生成。存储桶位于 AWS `us-east-1` 区域。

**AWS CLI 使用示例**

```
$ export AWS_ACCESS_KEY_ID={accessKeyId}
$ export AWS_SECRET_ACCESS_KEY={secretAccessKey}
$ export AWS_SESSION_TOKEN={sessionToken}
$ aws s3 cp /path/to/file s3://{bucket}/{key} --region us-east-1
```

### 创建上传

```endpoint
POST /uploads/v1/{username} uploads:write
```

使用临时 S3 凭证将文件保存到 Mapbox 的临时存储桶后，可以使用文件的 URL 和目标 tileset ID 生成 tileset。

上传的文件必须位于 Mapbox 提供的存储桶中。其他 S3 存储桶或 URL 的资源请求将会失败。

URL 参数 | 描述
--- | ---
`username` | The username of the account to which you are uploading上传帐户的用户名。

**请求体**

请求体必须是包含以下属性的JSON对象：

属性 | 描述
---- | ------
`tileset` | 创建或替换的地图 ID，格式： `username.nameoftileset`。限制为 32 个字符. 不可包含用户名， 特殊字符仅支持 `-` 和 `_`。
`url` | 凭据请求中提供的 S3 对象的 HTTPS URL，或要上传的现有 Mapbox 数据集的[数据集 ID](https://www.mapbox.com/help/define-dataset-id/) 。
`name`<br /> (可选) | tileset名称，限制为64个字符。

如果 `tileset` 值已存在，此操作将替换现有数据。使用随机值确保创建新的 tileset，或先检查现有的 tileset。

#### 请求示例

```curl
curl -X POST -H "Content-Type: application/json" -H "Cache-Control: no-cache" -d '{
  "url": "http://{bucket}.s3.amazonaws.com/{key}",
  "tileset": "{username}.{tileset-name}"
}' 'https://api.mapbox.com/uploads/v1/{username}?access_token=secret_access_token'
```

```javascript
// 调用 createUploadCredentials 的响应
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
    # Acquisition of credentials, staging of data, and upload
    # finalization is done by a single method in the Python SDK.
    upload_resp = service.upload(src, '{username}.data')
```

```bash
mapbox upload username.data data.geojson
```

```java
// 此 API 不支持 Mapbox Java SDK
```

```objc
// 此 API 不支持 Mapbox Objective-C 库
```

```swift
// 此 API 不支持 Mapbox Swift 库
```

#### 上传 Mapbox 数据集的请求体示例（不需要 AWS S3 存储桶）

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

**返回内容**

返回的内容是一个包含以下属性的JSON对象

属性 | 描述
---- | -----------
`complete` | 判断“upload”是否完整，完整则为“true”，否则为“false”
`tileset` | 如果该upload成功了，tileset的ID将会被创建或替换
`error` | 如果结果为“null”，表示upload正在执行或者已经成功，否则提示一个简洁的错误解释
`id` | upload的唯一标识
`name` | upload的名称
`modified` | upload最后一次被修改的时间戳
`created` | upload被创建的时间戳
`owner` | 用户账号的唯一标识
`progress` | 通过一个0和1之间的小数描述的upload的进程状态

### 获取 upload 状态

```endpoint
GET /uploads/v1/{username}/{upload_id} uploads:read
```

Upload 进程很快但不是瞬间完成的，每一次创建upload，你都可以追踪他的状态。当一个upload完成后，它都会有一个“progress”参数（介于0和1之间的小数）。假如一个upload中产生了错误，那么“error”属性将会包含一个[error message](https://www.mapbox.com/help/uploads/#errors)。

URL 参数 | 描述
--- | ---
`username` | 你所请求upload状态的账号的用户名
`upload_id` | upload的ID（假设你在请求一个成功upload）

**返回内容**

返回的内容是一个包含以下属性的JSON对象

属性 | 描述
---- | -----------
`complete` | 判断“upload”是否完整，完整则为“true”，否则为“false”
`tileset` | 如果该upload成功了，tileset的ID将会被创建或替换
`error` | 如果结果为“null”，表示upload正在执行或者已经成功，否则提示一个简洁的错误解释
`id` | upload的唯一标识
`name` | upload的名称
`modified` | upload最后一次被修改的时间戳
`created` | upload被创建的时间戳
`owner` | 用户账号的唯一标识
`progress` | 通过一个0和1之间的小数描述的upload的进程状态

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

### 获取最近的upload状态

```endpoint
GET /uploads/v1/{username} uploads:list
```

一次获取多个按照最近创建时间顺序排列的upload状态，存放在列表。这种请求返回的信息和一次单独的upload状态请求类似，但是对于同一个账户，这个列表的大小不能大于1MB

因为端点支持[pagination](#pagination)，所以你可以在列表中存放很多upload

URL 参数 | 描述
--- | ---
`username` | 你所请求upload状态的账号的用户名

从这个端点获得结果将能够通过如下参数进行修改

Query 参数 | 描述
--- | ---
`reverse`<br>(optional) | 设置为“true”将结果列表的默认排列顺序颠倒。
`limit`<br>(optional) | 返回状态数量的最大值，最大为“100”。

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

### 删除一个upload状态

```endpoint
DELETE /uploads/v1/{username}/{upload_id} uploads:write
```

从upload列表中删除一个完整upload的状态。

upload里只有状态（statue），因此从列表中删除upload并不会删除相关联的瓦片集。瓦片集只能通过Mapbox工作台删除。另外，一个upload的状态只有完整以后才能从upload列表中删除。

URL 参数 | 描述
--- | ---
`username` | 相关账户的用户名
`upload_id` | upload的ID（假设你在请求一个成功upload）

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
