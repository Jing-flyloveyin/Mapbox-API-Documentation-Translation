## Uploads

The Mapbox Uploads API transforms geographic data into tilesets that can be used with maps and geographic applications. Given a wide variety of geospatial formats, it normalizes projections and generates tiles at multiple zoom levels to make data viewable on the web.

The upload workflow follows these steps:

1. Request temporary S3 credentials that allow you to stage the file. _Jump to the [Retrieve S3 credentials  section](#retrieve-s3-credentials)._
1. Use an S3 client to upload the file to Mapbox's S3 staging bucket using these credentials. _Learn more about this process in the [Upload to Mapbox using cURL tutorial](https://www.mapbox.com/help/upload-curl/)._
1. Create an upload using the staged file's URL. _Jump to the [Create an upload section](#create-an-upload)._
1. Retrieve the upload's status as it is being processed into a tileset. _Jump to the [Retrieve upload status  section](#retrieve-upload-status)._

**Note:** This documentation discusses how to interact with the Mapbox Uploads API programatically. For a step-by-step guide on how to use the Uploads API to stage a file to Amazon S3 and create an upload, use the [Upload to Mapbox using cURL tutorial](https://www.mapbox.com/help/upload-curl/).

**Restrictions and limits**

- Tileset names are limited to 64 characters.
- The Uploads API supports different file sizes for various file types:

File type | Size limit
--- | ---
TIFF and GeoTIFF | 10 GB
MBTiles | 25 GB
GeoJSON | 1 GB
CSV | 1 GB
KML | 260 MB
GPX | 260 MB
Shapefile (zipped) | 260 MB
Mapbox Dataset | 1 GB

**Error messages**

To see a list of possible upload errors, visit the [Uploads errors page](https://www.mapbox.com/help/uploads/#errors).

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

Mapbox provides an Amazon S3 bucket to stage your file while your upload is processed. This endpoint allows you to retrieve temporary S3 credentials to use during the staging process.

**Note:** This step is necessary before you can stage a file in the Amazon S3 bucket provided by Mapbox. All uploads must be staged in this Amazon S3 bucket before being uploaded to your Mapbox account. To learn more about how to stage an upload to Amazon S3, read the [Upload to Mapbox using cURL tutorial](https://www.mapbox.com/help/upload-curl/#stage-your-file-on-amazon-s3).

URL parameters | Description
--- | ---
`username` | The username of the account to which you are uploading a tileset.

#### Example request

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
# AWS cannot be accessed through Mapbox CLI
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
  "accessKeyId": "{accessKeyId}",
  "bucket": "somebucket",
  "key": "hij456",
  "secretAccessKey": "{secretAccessKey}",
  "sessionToken": "{sessionToken}",
  "url": "{url}"
}
```

**Response body**

The response body is a JSON object that contains the following properties:

Property | Description
---- | -----------
`accessKeyId` | AWS Access Key ID
`bucket` | S3 bucket name
`key` | The unique key for data to be staged
`secretAccessKey` | AWS Secret Access Key
`sessionToken` | A temporary security token
`url` | The destination URL of the file

Use these credentials to store your data in the provided bucket with the provided key using the [AWS CLI](http://docs.aws.amazon.com/cli/latest/reference/s3/index.html) or AWS SDK of your choice. The bucket is located in AWS region `us-east-1`.

**Example AWS CLI usage**

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

After you have used the temporary S3 credentials to transfer your file to Mapbox's staging bucket, you can trigger the generation of a tileset using the file's URL and a destination tileset ID.

Uploaded files _must_ be in the bucket provided by Mapbox. Requests for resources from other S3 buckets or URLs will fail.

URL parameters | Description
--- | ---
`username` | The username of the account to which you are uploading

**Request body**

The request body must be a JSON object that contains the following properties:

Property | Description
---- | ------
`tileset` | The map ID to create or replace, in the format `username.nameoftileset`. Limited to 32 characters. This character limit does not include the username. The only allowed special characters are `-` and `_`.
`url` | The HTTPS URL of the S3 object provided in the credential request, or the [dataset ID](https://www.mapbox.com/help/define-dataset-id/) of an existing Mapbox dataset to be uploaded.
`name`<br /> (optional) | The name of the tileset. Limited to 64 characters.

If you reuse a `tileset` value, this action will replace existing data. Use a random value to make sure that a new tileset is created, or check your existing tilesets first.

#### Example request

```curl
curl -X POST -H "Content-Type: application/json" -H "Cache-Control: no-cache" -d '{
  "url": "http://{bucket}.s3.amazonaws.com/{key}",
  "tileset": "{username}.{tileset-name}"
}' 'https://api.mapbox.com/uploads/v1/{username}?access_token=secret_access_token'
```

```javascript
// Response from a call to createUploadCredentials
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
// This API cannot be accessed with the Mapbox Java SDK
```

```objc
// This API cannot be accessed with the Mapbox Objective-C libraries
```

```swift
// This API cannot be accessed with the Mapbox Swift libraries
```

#### Example request body to upload a Mapbox dataset (AWS S3 bucket not required)

```json
{
  "tileset": "{username}.mytileset",
  "url": "mapbox://datasets/{username}/{dataset}",
  "name": "example-dataset"
}
```

#### Example response

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
