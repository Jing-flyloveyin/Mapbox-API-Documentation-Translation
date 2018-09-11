## 令牌

一个 [访问令牌](#access-tokens), 以下简称为 '令牌', 代表用户授予对Mapbox资源的访问权. 所有的帐户在默认情况下都有一个公共令牌. 可以创建额外的令牌来授予额外的, 或者是更加受限的特权.

若要使用此API创建附加令牌，您将首先需要创建初始令牌. 此初始令牌必须包含 `tokens:write` 作用域和所有您想添加到创建的令牌的所有作用域. 为了创建初始令牌你需要访问 [Account Dashboard](https://www.mapbox.com/account/), 并且 **Create a token**.

```javascript
const mbxTokens = require('@mapbox/mapbox-sdk/services/tokens');
const tokensClient = mbxTokens({ accessToken: '{your_access_token}' });
```

### 令牌格式

Mapbox 使用 [JSON Web Tokens](https://jwt.io/) (JWT) 作为令牌的格式. 每个令牌是一个由点划分为三个部分的字符串：标题、有效载荷和签名.

 **标题** `pk` (公共令牌), `sk` (私密令牌), 或者 `tk` (临时令牌)的文字值.

**有效荷载** 是一个 base64-encoded JSON 对象 包含令牌的标识和权限. `pk`和 `sk` 令牌包含一个引用元数据的引用,该元数据拥有令牌授予的权限. `tk` 令牌 包含直接在有效载荷中的元数据的内容.

 **签名** 由Mapbox签名并用来验证令牌没有被篡改.

令牌允许的动作是基于**作用域 **. 作用域是一个字符串，通常是一个资源类型和一个由冒号分隔的动作. 例如,  `styles:read` 作用域允许对样式进行访问. 取决于他们的帐户级别和账户的特征,令牌有权访问不同的范围.

### 令牌元数据对象

每个令牌都有一个元数据对象，其中包含有关令牌功能的信息. 令牌元数据包含以下属性:

属性 | 描述
--- | ---
`id` | 令牌的标识符
`usage` | 令牌的类型
`client` | 客户端的令牌, 'api'
`default` | 如果令牌是默认令牌
`scopes` | 授予令牌的作用域数组
`note` | 令牌的人性化描述
`created` | 创建令牌的日期和时间
`modified` | 上次修改令牌的日期和时间
`token` | 令牌本身

#### 令牌元数据对象

```json
{
  "id": "cijucimbe000brbkt48d0dhcx",
  "usage": "pk",
  "client": "api",
  "default": false,
  "note": "My website",
  "scopes": ["styles:read", "fonts:read"],
  "created": "2016-01-25T19:07:07.621Z",
  "modified": "2016-01-26T00:39:57.941Z",
  "token": "pk.eyJ1Ijoic2NvdGhpcyIsImEiOiJjaWp1Y2ltYmUwMDBicmJrdDQ4ZDBkaGN4In0.sbihZCZJ56-fsFNKHXF8YQ"
}
```

**限制**

-请求必须通过HTTPS进行。不支持HTTP.
-令牌API仅限于每分钟120个请求.

### 列出令牌

列出帐户的所有令牌。.

这个端点支持 [pagination](#pagination).

```endpoint
GET /tokens/v2/{username} tokens:read
```

#### 实例请求

```curl
curl "https://api.mapbox.com/tokens/v2/{username}?access_token={your_access_token}"
```

```bash
# This API cannot be accessed with the Mapbox CLI
```

```javascript
tokensClient
  .listTokens()
  .send()
  .then(response => {
    const tokens = response.body;
  });
```

```python
# This API cannot be accessed with the Mapbox Python SDK
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

#### 实例响应

```json
[
  {
    "client": "api",
    "note": "a public token",
    "usage": "pk",
    "id": "cijucimbe000brbkt48d0dhcx",
    "default": false,
    "scopes": ["styles:read", "fonts:read"],
    "created": "2016-01-25T19:07:07.621Z",
    "modified":"2016-01-26T00:39:57.941Z",
    "token": "pk.eyJ1Ijoic2NvdGhpcyIsImEiOiJjaWp1Y2ltYmUwMDBicmJrdDQ4ZDBkaGN4In0.sbihZCZJ56-fsFNKHXF8YQ"
  },
  {
    "client": "api",
    "note": "a secret token",
    "usage": "sk",
    "id": "cijuorumy001cutm5r4fl2y1b",
    "default": false,
    "scopes": ["styles:list"],
    "created": "2016-01-26T00:50:13.701Z",
    "modified": "2016-01-26T00:50:13.701Z"
  }
]
```

Note: The token property will not be included for `sk` tokens.

### 创建令牌

Creates a new token.

Every requested scope must be present in the access token used to allow the request. It is not possible to create a token with access to more scopes than the token that created it.

The scopes included in the token decide whether the token is public or secret. A public token may only contains public scopes.

```endpoint
POST /tokens/v2/{username} tokens:write
```

#### 实例请求

```curl
curl -X POST "https://api.mapbox.com/tokens/v2/{username}?access_token={your_access_token}"
```

```bash
# This API cannot be accessed with the Mapbox CLI
```

```javascript
tokensClient
  .createToken({
    note: 'datasets-token',
    scopes: ['datasets:write', 'datasets:read']
  })
  .send()
  .then(response => {
    const token = response.body;
  });
```

```python
# This API cannot be accessed with the Mapbox Python SDK
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

#### 实例请求主体

```json
{
  "scopes": ["styles:read", "fonts:read"],
  "note": "My top secret project"
}
```

#### 实例响应

```json
{
  "client": "api",
  "note": "My top secret project",
  "usage": "pk",
  "id": "cijucimbe000brbkt48d0dhcx",
  "default": false,
  "scopes": ["styles:read", "fonts:read"],
  "created": "2016-01-25T19:07:07.621Z",
  "modified":"2016-01-25T19:07:07.621Z",
  "token": "pk.eyJ1Ijoic2NvdGhpcyIsImEiOiJjaWp1Y2ltYmUwMDBicmJrdDQ4ZDBkaGN4In0.sbihZCZJ56-fsFNKHXF8YQ"
}
```

### 创建临时令牌

Creates a new temporary token that automatically expires at a fixed time.

The `expires` property for a temporary token cannot be in the past or more than 1 hour in the future. If the authorizing token is temporary, the `expires` cannot be later than that of the authorizing token.

Every requested scope must be present in the authorizing token used to allow the request. It is not possible to create a token with access to more scopes than the token that created it.

Unlike public and secret tokens, a temporary token contains its metadata inside the payload of the token instead of referencing a metadata object that persists on the server. Temporary tokens cannot be updated or revoked after they are created.

```endpoint
POST /tokens/v2/{username} tokens:write
```

#### 实例请求

```curl
curl -X POST "https://api.mapbox.com/tokens/v2/{username}?access_token={your_access_token}"
```

```bash
# This API cannot be accessed with the Mapbox CLI
```

```javascript
tokensClient
  .createTemporaryToken({
    scopes: ['datasets:write', 'datasets:read']
  })
  .send()
  .then(response => {
    const token = response.body;
  });
```

```python
# This API cannot be accessed with the Mapbox Python SDK
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

#### 实例请求主体

```json
{
  "expires": "2016-09-15T19:27:53.000Z",
  "scopes": ["styles:read", "fonts:read"]
}
```

#### 实例响应

```json
{
  "token": "tk.eyJ1IjoibWFwYm94IiwiZXhwIjoxNDczOTY3NjczLCJpYXQiOjE0NzM5Njc2NDMsInNjb3BlcyI6WyJzdHlsZXM6cmVhZCIsImZvbnRzOnJlYWQiXSwiY2xpZW50IjoiYXBpIn0.ZepsWvpjTMlpePE4IQBs0g"
}
```

### 更新令牌
Update note or scopes in a token's metadata.

A public, `pk`, token may only be updated to include other public scopes. A secret, `sk`, token may be updated to contain public and secret scopes.

When updating scopes for an existing token, the token sent along with the request must also have the scopes you're requesting. It is not possible to create a token with access to more scopes than the token that updated it.

```endpoint
PATCH /tokens/v2/{username}/{token_id} tokens:write
```

#### 实例请求

```curl
curl -X PATCH  "https://api.mapbox.com/tokens/v2/{username}/{token_id}?access_token={your_access_token}"
```

```bash
# This API cannot be accessed with the Mapbox CLI
```

```javascript
tokensClient
  .updateToken({
    tokenId: 'cijucimbe000brbkt48d0dhcx',
    note: 'datasets-token',
    scopes: ['datasets:write', 'datasets:read']
  })
  .send()
  .then(response => {
    const token = response.body;
  });
```

```python
# This API cannot be accessed with the Mapbox Python SDK
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

#### 实例请求主体

```json
{
  "scopes": ["styles:tiles", "styles:read", "fonts:read"]
}
```

#### 实例响应

```json
{
  "client": "api",
  "note": "My top secret project",
  "usage": "pk",
  "id": "cijucimbe000brbkt48d0dhcx",
  "default": false,
  "scopes": ["styles:tiles", "styles:read", "fonts:read"],
  "created": "2016-01-25T19:07:07.621Z",
  "modified":"2016-01-25T19:07:07.621Z",
  "token": "pk.eyJ1Ijoic2NvdGhpcyIsImEiOiJjaWp1Y2ltYmUwMDBicmJrdDQ4ZDBkaGN4In0.sbihZCZJ56-fsFNKHXF8YQ"
}
```

### 删除令牌

Revoke a token's authorization, removing its access to Mapbox APIs. This is the same as deleting a token. Applications using the revoked token will need to get a new access token before they can access Mapbox APIs.

Note: Cached resources may continue to be accessible for a short period after a token is revoked. No new or updated resources will be accessible with the revoked token.

```endpoint
DELETE /tokens/v2/{username}/{token_id} tokens:write
```

#### 实例请求

```curl
curl -X DELETE "https://api.mapbox.com/tokens/v2/{username}/{token_id}?access_token={your_access_token}"
```

```bash
# This API cannot be accessed with the Mapbox CLI
```

```javascript
tokensClient
  .deleteToken({
    tokenId: 'cijucimbe000brbkt48d0dhcx'
  })
  .send()
  .then(response => {
    // Token successfully deleted.
  });
```

```python
# This API cannot be accessed with the Mapbox Python SDK
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

#### 实例响应

> HTTP 204

### 检索令牌

Check if a token is valid. If the token is invalid an explanation of why is returned as the `code` property.

The body of the token is parsed and included as the `token` property in object form.

Code | Description
--- | ---
`TokenValid` | the token is valid and active
`TokenMalformed` | the token can not be parsed
`TokenInvalid` | the signature for the token does not validate
`TokenExpired` | the token was temporary and expired
`TokenRevoked` | the token's authorization has been revoked

```endpoint
GET /tokens/v2?access_token={your_access_token}
```

#### 实例请求

```curl
$ curl "https://api.mapbox.com/tokens/v2?access_token={your_access_token}"
```

```bash
# This API cannot be accessed with the Mapbox CLI
```

```javascript
// Gets data about whatever token you've created the client with.
tokensClient
  .getToken()
  .send()
  .then(response => {
    const token = response.body;
  });
```

```python
# This API cannot be accessed with the Mapbox Python SDK
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

#### 公共令牌的实例响应

```json
{
  "code": "TokenValid",
  "token": {
    "usage": "pk",
    "user": "mapbox",
    "authorization": "cijucimbe000brbkt48d0dhcx"
  }
}
```

#### 临时令牌的实例响应

```json
{
  "code": "TokenExpired",
  "token": {
    "usage": "tk",
    "user": "mapbox",
    "expires": "2016-09-15T19:27:53.000Z",
    "created": "2016-09-15T19:27:23.000Z",
    "scopes": ["styles:read", "fonts:read"],
    "client": "api"
  }
}
```

### 列出作用域

为用户列出作用域. 列出了用户可以访问的所有可能作用域.

公共令牌只包含了 `public` 属性的作用域  `true`. 私密令牌可以包含任何作用域.

属性 | 描述
--- | ---
`id` | 作用域的标识符
`description` | 人性化的作用域描述
`public` | `true` 如果该作用域可用于公共令牌

```endpoint
GET /scopes/v1/{username}?access_token={your_access_token} scopes:list
```

#### 实例请求

```curl
$ curl "https://api.mapbox.com/scopes/v1/{username}?access_token={your_access_token}"
```

```bash
# This API cannot be accessed with the Mapbox CLI
```

```javascript
tokensClient
  .listScopes()
  .send()
  .then(response => {
    const scopes = response.body;
  });
```

```python
# This API cannot be accessed with the Mapbox Python SDK
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

#### 实例响应（截断）

```json
[
  {
    "id": "scopes:list",
    "description": "List all available scopes."
  },
  {
    "id": "styles:read",
    "public": true,
    "description": "Read styles."
  }
]
```
