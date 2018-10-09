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

URL 参数 | 描述
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
