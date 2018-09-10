## 样式

通过Mapbox 样式API，你可以读取和更改地图的样式、字体和图像。 样式API是[Mapbox Studio](https://www.mapbox.com/mapbox-studio/)的基础。

如果你使用Studio、[Mapbox GL JS](https://www.mapbox.com/mapbox-gl-js/api/)或[Mapbox Mobile SDKs](https://www.mapbox.com/mobile/)，那么你已经用到了样式API。这份文档适用于开发人员以编程方式读取写入这些资源。对于设计和使用Mapbox地图，你没必要读懂这份API文档。

为了更好使用样式API，你需要熟悉[Mapbox Style Specification](https://www.mapbox.com/mapbox-gl-style-spec)。Mapbox样式规范定义了地图样式的结构，并且是开放的标准，能够实现Studio和的APIs通信，生成可以与Mapbox库兼容的地图。 

**Mapbox 样式**

所有账户都可以使用有效token访问以下Mapbox样式：

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

导航样式功能参考[geographies that Mapbox supports](./pages/traffic-countries.html).

**限制和约束**

- 样式不能引用超过15个来源。
- 样式文件不能大于5M。 此约束只是适用于文件本身，对图标、字体、瓦片或其他参考资源没有限制。
- An account is allowed to have an unlimited number of styles regardless of its 无论哪种[pricing plan](http://mapbox.com/plans)，每个账户都有不限数量的地图样式。

```javascript
const mbxStyles = require('@mapbox/mapbox-sdk/services/styles');
const stylesClient = mbxStyles({ accessToken: '{your_access_token}' });
```

### 样式对象

样式对象在符合[Mapbox Style Specification](https://www.mapbox.com/mapbox-gl-style-spec)的同时, 还有与账户相关的属性:

属性 | 描述
--- | ---
`version` | 样式规范的版本号。
`name` | 样式的名称。
`metadata` | 有关Mapbox Studio使用该的样式的信息。
`sources` | 将会显示地图上的数据来源。
`layers` | 图层将会按照该数组的顺序绘制。
`created` | 样式创建的日期和时间。
`id` | 样式的ID。
`modified` | 样式最近一次修改日期和时间。
`owner` | 样式拥有者的名称。
`visibility` | 设置`公开`或`私有`的访问权限。 私有样式要求使用创建者的有效token访问，公开样式可以使用任何账户的有效token访问。
`draft` | 指示该样式是草稿 (`true`) ，还是已经发布了(`false`)。

**草稿**

样式 API 支持草稿，因此每种样式都可以有发布和草稿版本。这意味着你可以更改一个样式，但是不发布或部署到你的应用上。对于每一个样式相关端点，你都可以通过样式ID后添加`draft/`方式，切换到草稿版本，像`/styles/v1/{username}/{style_id}/draft/sprite`.

#### 样式对象例子

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

### 样式检索

```endpoint
GET /styles/v1/{username}/{style_id} styles:read
```

以JSON方式检索一个样式， 返回的[样式对象](#the-style-object)将符合[Mapbox样式](https://www.mapbox.com/mapbox-gl-style-spec/)的格式.

URL 参数 | 描述
-- | ---
`username` | 样式拥有者的账户名称。
`style_id` | 检索的样式ID。

#### 请求样例

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
# 此API不适用于 Python SDK
```

```bash
# 此API不适用于 Mapbox CLI
```

```java
// 此API不适用于 Mapbox Java SDK
```

```objc
// 此API不适用于 Mapbox Objective-C 库
// 可以使用Mapbox iOS SDK 或 Mapbox macOS SDK 代替
```

```swift
// 此API不适用于 Mapbox Swift 库
// 可以使用Mapbox iOS SDK 或 Mapbox macOS SDK 代替
```

#### 返回样例

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

### 样式列表

```endpoint
GET /styles/v1/{username} styles:list
```

检索某个指定账户的样式列表， 此请求返回样式的metadata，而不是整个样式信息。

URL 参数 | 描述
-- | ---
`username` | 样式拥有者的账户名称。

#### 请求样例

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
#  此API不适用于Python SDK
```

```bash
#  此API不适用于 Mapbox CLI
```

```java
// 此API不适用于Mapbox Java 库
```

```objc
// 此API不适用于Mapbox Objective-C 库
```

```swift
// 此API不适用于Mapbox Swift 库
```

#### 返回样例

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

### 样式创建

```endpoint
POST /styles/v1/{username} styles:write
```

用你的账户创建一个样式。提交的样式对象必须是JSON，且符合最新[Mapbox Style Specification](https://www.mapbox.com/mapbox-gl-style-spec/)。无效的样式对象将会导致描述性验证错误。

样式API不支持[GeoJSON sources](https://www.mapbox.com/mapbox-gl-js/style-spec/#sources-geojson)。

如果请求时，没有指定[可选 `name` 属性](https://www.mapbox.com/mapbox-gl-style-spec/#root-name)， 新样式的`name`将会默认设置为样式的ID。

API请求返回的样式包括了服务端添加的新的属性：`created`、`id`、`modified`、`owner`和`draft`。

URL 参数 | 描述
-- | ---
`username` | 样式拥有者的账户名称。

#### 请求样例

```curl
curl -X POST "https://api.mapbox.com/styles/v1/{username}?access_token={your_access_token}" \
  --data @basic-v9.json \
  --header "Content-Type:application/json"
```

```javascript
// 定义样式
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
# 此API不适用于Python SDK
```

```bash
此API不适用于Mapbox CLI
```

```java
// 此API不适用于Java SDK
```

```objc
// 此API不适用于Mapbox Objective-C库
```

```swift
// 此API不适用于Mapbox Swift库
```

#### 请求样例

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

#### 返回样例

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

### 样式更新

```endpoint
PATCH /styles/v1/{username}/{style_id} styles:write
```

更新账户里已有的样式。

你如果请求一个样式，然后用的相同的内容去更新该样式，你将会失败。在更新一个样式之前，你必须删除样式的`created`和`modified`属性。更新样式必须指定`name`属性, 但在[样式创建](#create-a-style)过程是可选的。

不允许跨版本的`PATCH`请求。

URL 参数 | 描述
--- | ---
`username` | 样式拥有者的账户名称。
`style_id` | 更新样式的ID。

#### 请求样例

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
```

```swift
// This API cannot be accessed with the Mapbox Swift libraries
```

#### Example request body

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

#### Example response

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

### Delete a style

```endpoint
DELETE /styles/v1/{username}/{style_id} styles:write
```

Delete a style. All sprites that belong to this style will also be deleted, and the style will no longer be available.

URL parameters | Description
--- | ---
`username` | The username of the account to which the style belongs.
`style_id` | The ID of the style to be deleted.

#### Example request

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

#### Response

> HTTP 204

### Request embeddable HTML

```endpoint
GET /styles/v1/{username}/{style_id}.html styles:read
```

Request embeddable HTML. The results can be displayed as a fullscreen map or can be inserted into `<iframe>` content.

URL parameters | Description
--- | ---
`username` | The username of the account to which the style belongs.
`style_id` | The ID of the style to be embedded.

The embeddable HTML that is returned can be further modified with the following optional query parameters:

Query Parameter | Description
----------|----------------------
`zoomwheel`<br /> (optional) | Whether to provide a zoomwheel, which enables a viewer to zoom in and out of the map using the mouse (`true`, default), or not (`false`).
`title`<br /> (optional) | Whether to display a title box with the map's title and owner in the upper right corner of the map (`true`) or not (`false`, default).
`fallback`<br /> (optional) | Serve a fallback raster map (`true`) or not (`false`, default).

#### Example style embed

```html
<iframe
  src='https://api.mapbox.com/styles/v1/mapbox/streets-v10.html?title=true&zoomwheel=false&access_token={your_access_token}' />
```

### Sprites

Sprites are the way that Mapbox GL JS and Mapbox Mobile efficiently request and show images. Sprites are collections of images that can be used in styles as icons or patterns in `symbol` layers. An image in a sprite can be an icon, a pattern, or an illustration. These SVG images can be added and removed from the sprite at will. The Styles API automatically collects these SVG images and renders them into a single PNG image and a JSON document that describes where each image is positioned.

The sprite JSON document is specified as part of the [the Mapbox Style Specification](https://www.mapbox.com/mapbox-gl-style-spec/#sprite).

Sprites are managed on a per-style basis. Each sprite belongs to a style, so the sprite limit of 500 images is also a per-style limit. All sprite-related API methods require a `{style_id}` parameter referring to the style to which the sprite belongs.

**Restrictions and limits**

- Each image must be smaller than 400kB.
- Mapbox supports most, but not all, SVG properties. These limits are described in our [SVG troubleshooting guide](https://www.mapbox.com/help/studio-troubleshooting-svg/).
- Images can be up to 512px in each dimension.
- Image names must be fewer than 255 characters in length.
- Sprites can contain up to 500 images.

### Retrieve a sprite image or JSON

```endpoint
GET /styles/v1/{username}/{style_id}/sprite{@2x}.{format} styles:read
```

Retrieve a sprite image or its JSON document from a Mapbox style.

URL Parameter | Description
----------|----------------------
`username` | The username of the account to which the style belongs.
`style_id` | The ID of the style to which the sprite belongs.
`@2x`<br /> (optional) | Render the sprite at a `@2x`, `@3x`, or `@4x` scale factor for high-density displays.
`format`<br /> (optional) | By default, this endpoint returns a sprite's JSON document. Specify `.png` to return the sprite image instead.

#### Example request

```curl
# Request the sprite image as a png
curl "https://api.mapbox.com/styles/v1/{username}/{style_id}/sprite.png?access_token={your_access_token}"

# Request json for a 3x scale sprite
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

#### Example response

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

### Add new image to sprite

```endpoint
PUT /styles/v1/{username}/{style_id}/sprite/{icon_name} styles:write
```

Add a new image to an existing sprite in a Mapbox style. The request body should be raw SVG data.

URL Parameter | Description
----------|----------------------
`username` | The username of the account to which the style belongs.
`style_id` | The ID of the style to which the sprite belongs.
`icon_name` | The name of the new image that is being added to the style.

#### Example request

```curl
# Add a new image (`aerialway`) to an existing sprite
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

#### Example response

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

### Delete image from sprite

```endpoint
DELETE /styles/v1/{username}/{style_id}/sprite/{icon_name} styles:write
```

Remove an image from an existing sprite.

URL Parameter | Description
----------|----------------------
`username` | The username of the account to which the style belongs.
`style_id` | The ID of the style to which the sprite belongs.
`icon_name` | The name of the new image to delete from the style.

#### Example request

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
```

```swift
// This API cannot be accessed with the Mapbox Swift libraries
```

#### Example response

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
