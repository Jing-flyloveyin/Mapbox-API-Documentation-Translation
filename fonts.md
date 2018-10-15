## 字体

Mapbox Fonts API 允许字体作为原始二进制数据，允许删除这些字体，并为地图渲染器生成编码字母。两种支持的字体类型：带有 `.ttf` 文件扩展名的TrueType字体，以及带有 `.otf` 扩展名的OpenType字体。

字体是基于单账户进行管理。样式可以使用同一个账户中的任意字体。

**约束和限制**

- 字体大小必须小于30MB。
- 一个账户最高100种字体

### 检索字体字形范围

```endpoint
GET /fonts/v1/{username}/{font}/{start}-{end}.pbf fonts:read
```

这是一个用来访问字体的端点，然而通常不会对字形范围感兴趣，除非你正在构建地图渲染器。

字体字形范围是缓冲区编码的有符号距离字段的协议。他们以各种比例大小和旋转角度来显示字体。一种字形可用于所有缩放比例。

URL Parameter | Description
--- | ---
`username` | 字体所属的帐户的用户名
`font` | 字体名称。该端点支持由逗号分隔的多个字体名称的查询。
`start` | 值取 `0` 到 `65280` 之间，`256` 的倍数
`end` |  `start` 的值加上 `255`.

**响应体**

成功的请求将返回 `HTTP 200 Success` 。响应体将是带有 `Content-Type：application / x-protobuf` 的字形缓冲区。

#### 请求例子

```curl
# 查询包含了2个用逗号分隔的字体名称
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
# 这个API不能通过 Python SDK 访问
```

```bash
这个API不能通过 Mapbox CLI 访问
```

```java
// 这个API不能通过 Mapbox Java SDK 访问
```

```objc
// 这个API不能通过 Mapbox Objective-C 库访问
```

```swift
// 这个API不能通过 Mapbox Swift 库访问
```


#### 响应

> `HTTP 200 Success`
