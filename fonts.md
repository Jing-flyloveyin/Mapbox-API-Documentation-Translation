## 字体

Mapbox字体API接受形式为原始二进制码的字体，允许删除这些字体，并且为地图渲染器生成代码。它支持两种类型的字体：TrueType字体，文件后缀名通常为`.ttf`，以及OpenType字体，后缀名为`.otf`。

字体按账户来管理。同一账户可以在样式中使用任何字体。

**限制和约束**

- 字体必须小于30MB。
- 每个账户最多使用100个字体。

### 检索字体字形范围

```endpoint
GET /fonts/v1/{username}/{font}/{start}-{end}.pbf fonts:read
```

虽然字形范围一般不被关注，除非你正在建立一个地图渲染器，但这是访问它们的端点。

字体字形范围是协议缓冲编码的带符号距离字段。它们可用于显示各种比例和旋转的字体。一个字形支持所有比例。

URL参数 | 描述
--- | ---
`username` | 该字体所属账户的用户名。
`font` | 字体的名称。此端点支持查询多个字体名称，名称间用逗号隔开。
`start` | `0`到`65280`之间`256`的倍数。
`end` | `start`显示的数字加上`255`。

**响应主体**

一个成功的请求将会返回`HTTP 200 Success`。响应主体将会是字形的一个缓存，字形带有`Content-Type: application/x-protobuf`。

#### 示例请求

```curl
# 查询包含两个由逗号分隔的字体名
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
# 无法使用Python SDK访问此API
```

```bash
无法使用Mapbox CLI访问此API
```

```java
// 无法使用Mapbox Java SDK访问此API
```

```objc
// 无法使用Mapbox Objective-C 库访问此API
```

```swift
// 无法使用Mapbox Swift库访问此API
```


#### 响应

> `HTTP 200 Success`
