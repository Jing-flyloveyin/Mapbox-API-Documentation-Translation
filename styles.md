## Styles

The Mapbox Styles API lets you read and change map styles, fonts, and images. This API is the basis for [Mapbox Studio](https://www.mapbox.com/mapbox-studio/).

If you use Studio, [Mapbox GL JS](https://www.mapbox.com/mapbox-gl-js/api/), or the [Mapbox Mobile SDKs](https://www.mapbox.com/mobile/), you are already using the Styles API. This documentation is meant for software developers who want to programmatically read and write these resources. It isn't necessary for you to read or understand this reference to design or use Mapbox maps.

You will need to be familiar with the [Mapbox Style Specification](https://www.mapbox.com/mapbox-gl-style-spec) to use the Styles API. The Mapbox Style Specification defines the structure of map styles and is the open standard that helps Studio communicate with APIs and produce maps that are compatible with Mapbox libraries.

**Mapbox styles**

The following Mapbox styles are available to all accounts using a valid access token:

- [`mapbox://styles/mapbox/streets-v10`](https://api.mapbox.com/styles/v1/mapbox/streets-v10.html?title=true&access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NDg1bDA1cjYzM280NHJ5NzlvNDMifQ.d6e-nNyBDtmQCVwVNivz7A#2/0.0/0.0)
- [`mapbox://styles/mapbox/outdoors-v10`](https://api.mapbox.com/styles/v1/mapbox/outdoors-v10.html?title=true&access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NDg1bDA1cjYzM280NHJ5NzlvNDMifQ.d6e-nNyBDtmQCVwVNivz7A#2/0.0/0.0)
- [`mapbox://styles/mapbox/light-v9`](https://api.mapbox.com/styles/v1/mapbox/light-v9.html?title=true&access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NDg1bDA1cjYzM280NHJ5NzlvNDMifQ.d6e-nNyBDtmQCVwVNivz7A#2/0.0/0.0)
- [`mapbox://styles/mapbox/dark-v9`](https://api.mapbox.com/styles/v1/mapbox/dark-v9.html?title=true&access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NDg1bDA1cjYzM280NHJ5NzlvNDMifQ.d6e-nNyBDtmQCVwVNivz7A#2/0.0/0.0)
- [`mapbox://styles/mapbox/satellite-v9`](https://api.mapbox.com/styles/v1/mapbox/satellite-v9.html?title=true&access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NDg1bDA1cjYzM280NHJ5NzlvNDMifQ.d6e-nNyBDtmQCVwVNivz7A#2/0.0/0.0)
- [`mapbox://styles/mapbox/satellite-streets-v10`](https://api.mapbox.com/styles/v1/mapbox/satellite-streets-v10.html?title=true&access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NDg1bDA1cjYzM280NHJ5NzlvNDMifQ.d6e-nNyBDtmQCVwVNivz7A#2/0.0/0.0)
- [`mapbox://styles/mapbox/navigation-preview-day-v4`](https://api.mapbox.com/styles/v1/mapbox/navigation-preview-day-v4.html?title=true&access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NDg1bDA1cjYzM280NHJ5NzlvNDMifQ.d6e-nNyBDtmQCVwVNivz7A#2/0.0/0.0)
- [`mapbox://styles/mapbox/navigation-preview-night-v4`](https://api.mapbox.com/styles/v1/mapbox/navigation-preview-night-v4.html?title=true&access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NDg1bDA1cjYzM280NHJ5NzlvNDMifQ.d6e-nNyBDtmQCVwVNivz7A#2/0.0/0.0)
- [`mapbox://styles/mapbox/navigation-guidance-day-v4`](https://api.mapbox.com/styles/v1/mapbox/navigation-guidance-day-v4.html?title=true&access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NDg1bDA1cjYzM280NHJ5NzlvNDMifQ.d6e-nNyBDtmQCVwVNivz7A#2/0.0/0.0)
- [`mapbox://styles/mapbox/navigation-guidance-night-v4`](https://api.mapbox.com/styles/v1/mapbox/navigation-guidance-night-v4.html?title=true&access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NDg1bDA1cjYzM280NHJ5NzlvNDMifQ.d6e-nNyBDtmQCVwVNivz7A#2/0.0/0.0)

Navigation styles feature coverage in [geographies that Mapbox supports](./pages/traffic-countries.html).

**Restrictions and limits**

- Styles cannot reference more than 15 sources.
- Styles cannot be larger than 5MB. This limit only applies to the style document itself, not the sprites, fonts, tilesets, or other resources it references.
- An account is allowed to have an unlimited number of styles regardless of its [pricing plan](http://mapbox.com/plans).

```javascript
const mbxStyles = require('@mapbox/mapbox-sdk/services/styles');
const stylesClient = mbxStyles({ accessToken: '{your_access_token}' });
```

### The style object

A style object is an object that conforms to the [Mapbox Style Specification](https://www.mapbox.com/mapbox-gl-style-spec), with some additional account-related properties:

Property | Description
--- | ---
`version` | The style specification version number.
`name` | A human-readable name for the style.
`metadata` | Information about the style that is used in Mapbox Studio.
`sources` | Sources supply the data that will be shown on the map.
`layers` | Layers will be drawn in the order of this array.
`created` | The date and time the style was created.
`id` | The ID of the style.
`modified` | The date and time the style was last modified.
`owner` | The username of the style owner.
`visibility` | Access control for the style, either `public` or `private`. Private styles require an access token belonging to the owner. Public styles may be requested with an access token belonging to any user.
`draft` | Indicates whether the style is a draft (`true`) or whether it has been published (`false`).

**About drafts**

The Styles API supports drafts, so every style can have both published and draft versions. This means that you can make changes to a style without publishing them or deploying them in your app. For each style-related endpoint, you can interact with the draft version of a style by placing `draft/` after the style ID, like `/styles/v1/{username}/{style_id}/draft/sprite`.

#### Example style object

```json
{
  "version": 8,
  "name": "{name}",
  "metadata": "{metadata}",
  "sources": "{sources}",
  "sprite": "mapbox://sprites/{username}/{style_id}",
  "glyphs": "mapbox://fonts/{username}/{fontstack}/{range}.pbf",
  "layers": ["{layers}"],
  "created": "2015-10-30T22:18:31.111Z",
  "id": "{style_id}",
  "modified": "2015-10-30T22:22:06.077Z",
  "owner": "{username}",
  "visibility": "private",
  "draft": true
}
```

### Retrieve a style

```endpoint
GET /styles/v1/{username}/{style_id} styles:read
```

Retrieve a style as a JSON document. The returned [style object](#the-style-object) will be in the [Mapbox Style](https://www.mapbox.com/mapbox-gl-style-spec/) format.

URL parameters | Description
-- | ---
`username` | The username of the account to which the style belongs.
`style_id` | The ID of the style to be retrieved.

#### Example request

```curl
curl "https://api.mapbox.com/styles/v1/examples/cjikt35x83t1z2rnxpdmjs7y7?access_token={your_access_token}"
```

```javascript
stylesClient
  .getStyle({
    styleId: 'style-id'
  })
  .send()
  .then(response => {
    const style = response.body;
  });
```

```python
# This API cannot be accessed with the Python SDK
```

```bash
# This API cannot be accessed with Mapbox CLI
```

```java
// This API cannot be accessed with the Mapbox Java SDK
```

```objc
// This API cannot be accessed with the Mapbox Objective-C libraries
// Use the Mapbox iOS SDK or Mapbox macOS SDK instead
```

```swift
// This API cannot be accessed with the Mapbox Swift libraries
// Use the Mapbox iOS SDK or Mapbox macOS SDK instead
```

#### Example response

```json
{
  "version": 8,
  "name": "Meteorites",
  "metadata": {
    "mapbox:origin": "basic-template-v1",
    "mapbox:autocomposite": true,
    "mapbox:type": "template",
    "mapbox:sdk-support": {
      "js": "0.45.0",
      "android": "6.0.0",
      "ios": "4.0.0"
    }
  },
  "center": [
    74.24426803763072,
    -2.2507114487818853
  ],
  "zoom": 0.6851443156248076,
  "bearing": 0,
  "pitch": 0,
  "sources": {
    "composite": {
      "url": "mapbox://mapbox.mapbox-streets-v7,examples.0fr72zt8",
      "type": "vector"
    }
  },
  "sprite": "mapbox://sprites/examples/cjikt35x83t1z2rnxpdmjs7y7",
  "glyphs": "mapbox://fonts/{username}/{fontstack}/{range}.pbf",
  "layers": [
    {
      "id": "background",
      "type": "background",
      "layout": {},
      "paint": {
        "background-color": [ ]
      }
    },
    { }
  ],
  "created": "2015-10-30T22:18:31.111Z",
  "id": "cjikt35x83t1z2rnxpdmjs7y7",
  "modified": "2015-10-30T22:22:06.077Z",
  "owner": "examples",
  "visibility": "public",
  "draft": false
}
```

### List styles

```endpoint
GET /styles/v1/{username} styles:list
```

Retrieve a list of styles for a specific account. This endpoint returns style metadata instead of returning full styles.

URL parameters | Description
-- | ---
`username` | The username of the account to which the styles belong.

#### Example request

```curl
curl "https://api.mapbox.com/styles/v1/{username}?access_token={your_access_token}"
```

```javascript
stylesClient
  .listStyles()
  .send()
  .then(response => {
    const styles = response.body;
  });
```

```python
# This API cannot be accessed with the Python SDK
```

```bash
# This API cannot be accessed with Mapbox CLI
```

```java
// This API cannot be accessed with Mapbox Java libraries
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
    "version": 8,
    "name": "My Awesome Style",
    "created": "{timestamp}",
    "id": "cige81msw000acnm7tvsnxcp5",
    "modified": "{timestamp}",
    "owner": "{username}"
  },
  {
    "version": 8,
    "name": "My Cool Style",
    "created": "{timestamp}",
    "id": "cig9rvfe300009lj9kekr0tm2",
    "modified": "{timestamp}",
    "owner": "{username}"
  }
]
```

### Create a style

```endpoint
POST /styles/v1/{username} styles:write
```

Creates a style in your account. The posted style object must be both valid JSON and aligned to the most recent version of the [Mapbox Style Specification](https://www.mapbox.com/mapbox-gl-style-spec/). Invalid styles will produce a descriptive validation error.

[GeoJSON sources](https://www.mapbox.com/mapbox-gl-js/style-spec/#sources-geojson) are not supported by the Styles API.

If the [optional `name` property](https://www.mapbox.com/mapbox-gl-style-spec/#root-name) is not used in the request body, the `name` of the new style will be automatically set to the style's ID.

The style you get back from the API will contain new properties that the server has added: `created`, `id`, `modified`, `owner`, and `draft`.

URL parameters | Description
-- | ---
`username` | The username of the account to which the new style will belong.

#### Example request

```curl
curl -X POST "https://api.mapbox.com/styles/v1/{username}?access_token={your_access_token}" \
  --data @basic-v9.json \
  --header "Content-Type:application/json"
```

```javascript
// Define the style
stylesClient
  .createStyle({
    style: {
      version: 8,
      name: "My Awesome Style",
      metadata: {},
      sources: {},
      layers: [],
      glyphs: "mapbox://fonts/{owner}/{fontstack}/{range}.pbf"
    }
  })
  .send()
  .then(response => {
    const style = response.body;
  });
```

```python
# This API cannot be accessed with the Python SDK
```

```bash
This API cannot be accessed with Mapbox CLI
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
  "version": 8,
  "name": "My Awesome Style",
  "metadata": { },
  "sources": {
    "myvectorsource": {
      "url": "mapbox://{map_id}",
      "type": "vector"
    },
    "myrastersource": {
      "url": "mapbox://{map_id}",
      "type": "raster"
    }
  },
  "glyphs": "mapbox://fonts/{username}/{fontstack}/{range}.pbf",
  "layers": [ ]
}
```

#### Example response

```json
{
  "version": 8,
  "name": "My Awesome Style",
  "metadata": { },
  "sources": {
    "myvectorsource": {
      "url": "mapbox://{map_id}",
      "type": "vector"
    },
    "myrastersource": {
      "url": "mapbox://{map_id}",
      "type": "raster"
    }
  },
  "sprite": "mapbox://sprites/{username}/{style_id}",
  "glyphs": "mapbox://fonts/{username}/{fontstack}/{range}.pbf",
  "layers": [ ],
  "created": "2015-10-30T22:18:31.111Z",
  "id": "{style_id}",
  "modified": "2015-10-30T22:22:06.077Z",
  "owner": "{username}",
  "draft": true
}
```

### Update a style

```endpoint
PATCH /styles/v1/{username}/{style_id} styles:write
```

Updates an existing style in your account with new content.

If you request a style and then use the same content to update the style, this action will fail. You must remove the `created` and `modified` properties before updating a style. The `name` property, which is optional for [creating a style](#create-a-style), is required to update a style.

Cross-version `PATCH` requests are rejected.

URL parameters | Description
--- | ---
`username` | The username of the account to which the style belongs.
`style_id` | The ID of the style to be updated.

#### Example request

```curl
curl -X PATCH "https://api.mapbox.com/styles/v1/{username}/{style_id}" \
  --data @basic-v9.json \
  --header "Content-Type:application/json"
```

```javascript
stylesClient
  .updateStyle({
    styleId: 'style-id',
    style: {
      version: 8,
      name: 'My Awesome Style',
      metadata: {},
      sources: {},
      layers: [],
      glyphs: 'mapbox://fonts/{owner}/{fontstack}/{range}.pbf'
    }
  })
  .send()
  .then(response => {
    const style = response.body;
  });
```

```python
# 用Python SDK不能进入API。
```

```bash
# 用Mapbox CLI不能进入API。
```

```java
// 用Mapbox Java SDK不能进入API。
```

```objc
// 用Mapbox Objective-C libraries不能进入API。
```

```swift
// 用Mapbox Swift libraries不能进入API。
```

#### 示例请求体

```json
{
  "version": 8,
  "name": "New Style Name",
  "metadata": { },
  "sources": { },
  "sprite": "mapbox://sprites/{username}/{style_id}",
  "glyphs": "mapbox://fonts/{username}/{fontstack}/{range}.pbf",
  "layers": [{
    "id": "new-layer",
    "type": "background",
    "paint": {
      "background-color": "#111"
    },
    "interactive": true
  }],
  "owner": "{username}",
  "draft": true
}
```

#### 示例反应

```json
{
  "version": 8,
  "name": "New Style Name",
  "metadata": { },
  "sources": { },
  "sprite": "mapbox://sprites/{username}/{style_id}",
  "glyphs": "mapbox://fonts/{username}/{fontstack}/{range}.pbf",
  "layers": [{
    "id": "new-layer",
    "type": "background",
    "paint": {
      "background-color": "#111"
    },
    "interactive": true
  }],
  "created": "2015-10-30T22:18:31.111Z",
  "id": "{style_id}",
  "modified": "2015-10-30T22:22:06.077Z",
  "owner": "{username}",
  "draft": true
}
```

### 删除格式

```endpoint
DELETE /styles/v1/{username}/{style_id} styles:write
```

删除格式。 同时也会删除该格式的所有子画面，格式将不存在。 
URL parameters | Description
--- | ---
`username` | 格式账户用户名。
`style_id` | 格式ID被删除。

#### 示例请求

```curl
curl -X DELETE "https://api.mapbox.com/styles/v1/{username}/{style_id}?access_token={your_access_token}"
```

```javascript
stylesClient
  .deleteStyle({
    styleId: 'style-id'
  })
  .send()
  .then(response => {
    // delete successful
  });
```

```python
# 用Python SDK不能进入API。
```

```bash
用Mapbox CLI不能进入API。
```

```java
//  用Mapbox Java SDK不能进入API。
```

```objc
// 用Mapbox Objective-C libraries不能进入API。
```

```swift
// 用Mapbox Swift libraries不能进入API。
```

#### 反应

> HTTP 204

### 请求可嵌入 HTML

```endpoint
GET /styles/v1/{username}/{style_id}.html styles:read
```

请求可嵌入 HTML。 结果可展示为全屏地图或插入`<iframe>` 内容。

URL 参数| 描述
--- | ---
`username` | 格式账户用户名。
`style_id` | 要嵌入的格式ID。

可用下述可选查询参数进一步修改可嵌入HTML。

查询参数 | 描述
----------|----------------------
`zoomwheel`<br /> (optional) | 用户是否可以切换鼠标滚轮缩放， 用户可用鼠标 (`true`, default), 或 (`false`)缩放地图。
`title`<br /> (optional) |是否在地图的右上角用地图标题和所有者展示标题 W(`true`) 或 (`false`, default).
`fallback`<br /> (optional) | 退回栅格地图 (`true`) 或 (`false`, default).

#### 嵌入的示例样式

```html
<iframe
  src='https://api.mapbox.com/styles/v1/mapbox/streets-v10.html?title=true&zoomwheel=false&access_token={your_access_token}' />
```

### 子画面

子画面是Mapbox GL JS和 Mapbox Mobile有效请求和展示图像的方式。 子画面是在“符号”层可用作图表或图样的图像集合。子画面中的图像可为图表、图样或插图。可在子画面中随时添加或删除这些SVG图像。样式API自动收集这些SVG图像并将其渲染成为单张PBG图像以及描述每张图像定位地点的JSON文件。

子画面 JSON 文件确定为  [the Mapbox Style Specification](https://www.mapbox.com/mapbox-gl-style-spec/#sprite)的部分。

按照每种样式管理子画面。每个子画面属于一个样式，因此样式的限制为500张图像，也是每种样式的限制。所有子画面相关的API方式需要`{style_id}`参数，该参数指子画面所属的样式。

**限制和条件**

- 每张图像必须小于 400kB。
- Mapbox 支持大部分SVG属性，而不是全部。 [SVG troubleshooting guide](https://www.mapbox.com/help/studio-troubleshooting-svg/)列出了这些限制。
-  图像每个维度可达512px 。
- 图像名称长度必须小于 255 字符。
- 子画面可含 500 张图像。

### 检索子画面图像或  JSON

```endpoint
GET /styles/v1/{username}/{style_id}/sprite{@2x}.{format} styles:read
```

 从Mapbox 样式中检索子画面图像或其JSON文件。

URL 参数| 描述
----------|----------------------
`username` | 样式账户用户名。
`style_id` | 子画面样式的ID。
`@2x`<br /> (optional) | 在 `@2x`, `@3x`, or `@4x` 比例因子渲染子画面， 高密度展示。
`format`<br /> (optional) | 默认情况下，终点返回子画面的 JSON 文件。制定 `.png` 返回子画面图像。

#### 示例请求

```curl
# 请求作为 png的子画面图像。
curl "https://api.mapbox.com/styles/v1/{username}/{style_id}/sprite.png?access_token={your_access_token}"

# 请求 json ， 一张 3x 比例的子画面
curl "https://api.mapbox.com/styles/v1/{username}/{style_id}/sprite@3x?access_token={your_access_token}"
```

```javascript
stylesClient
  .getStyleSprite({
    format: 'json',
    styleId: 'foo',
    highRes: true
  })
  .send()
  .then(response => {
    const sprite = response.body;
  });
```

```python
#  用Python SDK不能进入API。
```

```bash
用Mapbox CLI不能进入API。
```

```java
// 用Mapbox Java SDK不能进入API。
```

```objc
// 用Mapbox Objective-C libraries不能进入API。
```

```swift
// 用Mapbox Swift libraries不能进入API。
```

#### 示例响应

```json
{
  "default_marker": {
    "width": 20,
    "height": 50,
    "x": 0,
    "y": 0,
    "pixelRatio": 2
  },
  "secondary_marker": {
    "width": 20,
    "height": 50,
    "x": 20,
    "y": 0,
    "pixelRatio": 2
  }
}
```

### 在子画面中添加新图像

```endpoint
PUT /styles/v1/{username}/{style_id}/sprite/{icon_name} styles:write
```

在Mapbox样式中现有的子画面中添加新图像。 请求体应为 原SVG 数据。

URL 参数 |描述
----------|----------------------
`username` | 样式账户用户名。
`style_id` | 子画面样式ID。
`icon_name` | 添加到样式中新图像的名称。

#### 示例请求

```curl
# 将新图像 (`aerialway`) 添加到现有子画面中。
curl -X PUT \
  "https://api.mapbox.com/styles/v1/{username}/{style_id}/sprite/aerialway?access_token={your_access_token}" \
  --data @aerialway-12.svg
```

```javascript
stylesClient
  .putStyleIcon({
    styleId: 'foo',
    iconId: 'bar',
    // The string filename value works in Node.
    // In the browser, provide a Blob.
    file: 'path/to/file.svg'
  })
  .send()
  .then(response => {
    const newSprite = response.body;
  });
```

```python
#  用Python SDK不能进入API。
```

```bash
用Mapbox CLI不能进入API。
```

```java
// 用Mapbox Java SDK不能进入API。
```

```objc
//  用Mapbox Objective-C libraries不能进入API。
```

```swift
//  用Mapbox Swift libraries不能进入API。
```

#### 示例响应

```json
{
  "newsprite": {
    "width": 1200,
    "height": 600,
    "x": 0,
    "y": 0,
    "pixelRatio": 1
  },
  "default_marker": {
    "width": 20,
    "height": 50,
    "x": 0,
    "y": 600,
    "pixelRatio": 1
  }
}
```

### 从子画面中删除图像

```endpoint
DELETE /styles/v1/{username}/{style_id}/sprite/{icon_name} styles:write
```

从现有子画面中删掉图像。

URL 参数 | 描述
----------|----------------------
`username` | 样式账户的用户名。
`style_id` | 子画面样式的ID。
`icon_name` | 要从样式中删除的新图像名称。

#### 示例请求

```curl
curl -X DELETE "https://api.mapbox.com/styles/v1/{username}/{style_id}/sprite/{icon_name}?access_token={your_access_token}"
```

```javascript
stylesClient
  .deleteStyleIcon({
    styleId: 'foo',
    iconId: 'bar'
  })
  .send()
  .then(response => {
    // delete successful
  });
```

```python
# 用Python SDK不能进入API。
```

```bash
# 用Mapbox CLI不能进入API。
```

```java
//  用Mapbox Java SDK不能进入API。
```

```objc
// 用Mapbox Objective-C libraries不能进入API。
```

```swift
// 用Mapbox Swift libraries不能进入API。
```

#### 示例响应

```json
{
  "default_marker": {
    "width": 20,
    "height": 50,
    "x": 0,
    "y": 600,
    "pixelRatio": 1
  },
  "secondary_marker": {
    "width": 20,
    "height": 50,
    "x": 20,
    "y": 600,
    "pixelRatio": 1
  }
}
```
