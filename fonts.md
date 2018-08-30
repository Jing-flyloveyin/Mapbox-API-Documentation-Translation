## Fonts

The Mapbox Fonts API accepts fonts as raw binary data, allows those fonts to be deleted, and generates encoded letters for map renderers. Two types of fonts are supported: TrueType fonts, usually with `.ttf` file extensions, and OpenType fonts, with `.otf` extensions.

Fonts are managed on a per-account basis. Styles can use any font from the same account.

**Restrictions and limits**

- Fonts must be smaller than 30MB.
- Accounts are limited to 100 fonts.

### Retrieve font glyph ranges

```endpoint
GET /fonts/v1/{username}/{font}/{start}-{end}.pbf fonts:read
```

While glyph ranges are usually not of interest unless you're building a map renderer, this is the endpoint you can use to access them.

Font glyph ranges are protocol buffer-encoded signed distance fields. They can be used to show fonts at a variety of scales and rotations. One glyph is used at all scales.

URL Parameter | Description
--- | ---
`username` | The username of the account to which the font belongs.
`font` | The name of the font. This endpoint supports queries with multiple comma-separated font names.
`start` | A multiple of `256` between `0` and `65280`.
`end` | The number indicated by `start`, plus `255`.

**Response body**

A successful request will return `HTTP 200 Success`. The response body will be a buffer of the glyphs with `Content-Type: application/x-protobuf`.

#### Example request

```curl
# Query contains 2 comma-separated font names
curl "https://api.mapbox.com/fonts/v1/examples/DIN Offc Pro Medium,Arial Unicode MS Regular/0-255?access_token={your_access_token}"
```

```javascript
stylesClient
  .getFontGlyphRange({
    fonts: 'Arial Unicode',
    start: 0,
    end: 255
  })
  .send()
  .then(response => {
    const glyph = response.body;
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

> `HTTP 200 Success`
