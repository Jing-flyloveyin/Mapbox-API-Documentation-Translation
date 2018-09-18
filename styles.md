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

子画面是Mapbox GL JS和 Mapbox Mobile有效请求和展示图像的方式。子画面是在“符号”层可用作图表或图样的图像集合。子画面中的图像可为图表、图样或插图。可在子画面中随时添加或删除这些SVG图像。样式API自动收集这些SVG图像并将其渲染成为单张PBG图像以及描述每张图像定位地点的JSON文件。

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
