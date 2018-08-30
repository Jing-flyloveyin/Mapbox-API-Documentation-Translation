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
