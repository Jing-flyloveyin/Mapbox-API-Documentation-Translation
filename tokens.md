## Tokens

An [access token](#access-tokens), referred to hereafter as 'token', grants access to Mapbox resources on behalf of a user. All accounts have a public token by default. Additional tokens can be created to grant additional, or more limited, privileges.

To create additional tokens using this API you will first need to create an initial token. This initial token must contain the `tokens:write` scope and all scopes you want to add to the created token. To create the initial token visit your [Account Dashboard](https://www.mapbox.com/account/), and click **Create a token**.

```javascript
const mbxTokens = require('@mapbox/mapbox-sdk/services/tokens');
const tokensClient = mbxTokens({ accessToken: '{your_access_token}' });
```

### The token format

Mapbox uses [JSON Web Tokens](https://jwt.io/) (JWT) as the token format. Each token is a string delimited by dots into three parts: header, payload, and signature.

The **header** is a literal value of either `pk` (public token), `sk` (secret token), or `tk` (temporary token).

The **payload** is a base64-encoded JSON object containing the identity and authorities of the token. `pk` and `sk` tokens contain a reference to metadata which holds the rights granted for the token. `tk` tokens contain the content of the metadata directly in the payload.

The **signature** is signed by Mapbox and used to verify the token has not been tampered with.

The actions allowed by a token are based on **scopes**. A scope is a string that often is a resource type and action separated by a colon. For example, the `styles:read` scope allows read access to styles. Tokens will have access to different scopes depending on their account level and other features of their account.

### The token metadata object

Every token has a metadata object that contains information about the capabilities of the token. Token metadata contains the following properties:

Property | Description
--- | ---
`id` | the identifier for the token
`usage` | the type of token
`client` | the client for the token, always 'api'
`default` | if the token is a default token
`scopes` | array of scopes granted to the token
`note` | human friendly description of the token
`created` | date and time the token was created
`modified` | date and time the token was last modified
`token` | the token itself

#### The token metadata object

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

**Limits**

- Requests must be made over HTTPS. HTTP is not supported.
- The Tokens API is limited to 120 requests per minute per account.

### List tokens

Lists all tokens for an account.

This endpoint supports [pagination](#pagination).

```endpoint
GET /tokens/v2/{username} tokens:read
```

#### Example request

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

#### Example response

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

### Create token

Creates a new token.

Every requested scope must be present in the access token used to allow the request. It is not possible to create a token with access to more scopes than the token that created it.

The scopes included in the token decide whether the token is public or secret. A public token may only contains public scopes.

```endpoint
POST /tokens/v2/{username} tokens:write
```

#### Example request

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

#### Example request body

```json
{
  "scopes": ["styles:read", "fonts:read"],
  "note": "My top secret project"
}
```

#### Example response

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

### Create temporary token

Creates a new temporary token that automatically expires at a fixed time.

The `expires` property for a temporary token cannot be in the past or more than 1 hour in the future. If the authorizing token is temporary, the `expires` cannot be later than that of the authorizing token.

Every requested scope must be present in the authorizing token used to allow the request. It is not possible to create a token with access to more scopes than the token that created it.

Unlike public and secret tokens, a temporary token contains its metadata inside the payload of the token instead of referencing a metadata object that persists on the server. Temporary tokens cannot be updated or revoked after they are created.

```endpoint
POST /tokens/v2/{username} tokens:write
```

#### Example request

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

#### Example request body

```json
{
  "expires": "2016-09-15T19:27:53.000Z",
  "scopes": ["styles:read", "fonts:read"]
}
```

#### Example response

```json
{
  "token": "tk.eyJ1IjoibWFwYm94IiwiZXhwIjoxNDczOTY3NjczLCJpYXQiOjE0NzM5Njc2NDMsInNjb3BlcyI6WyJzdHlsZXM6cmVhZCIsImZvbnRzOnJlYWQiXSwiY2xpZW50IjoiYXBpIn0.ZepsWvpjTMlpePE4IQBs0g"
}
```

### Update a token

Update note or scopes in a token's metadata.

A public, `pk`, token may only be updated to include other public scopes. A secret, `sk`, token may be updated to contain public and secret scopes.

When updating scopes for an existing token, the token sent along with the request must also have the scopes you're requesting. It is not possible to create a token with access to more scopes than the token that updated it.

```endpoint
PATCH /tokens/v2/{username}/{token_id} tokens:write
```

#### Example request

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

#### Example request body

```json
{
  "scopes": ["styles:tiles", "styles:read", "fonts:read"]
}
```

#### Example response

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

### Delete a token

Revoke a token's authorization, removing its access to Mapbox APIs. This is the same as deleting a token. Applications using the revoked token will need to get a new access token before they can access Mapbox APIs.

Note: Cached resources may continue to be accessible for a short period after a token is revoked. No new or updated resources will be accessible with the revoked token.

```endpoint
DELETE /tokens/v2/{username}/{token_id} tokens:write
```

#### Example request

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

#### Example response

> HTTP 204

### Retrieve a token

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

#### Example request

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

#### Example response for a public token

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

#### Example response for a temporary token

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

### List scopes

List scopes for a user. All potential scopes a user has access to are listed.

Public tokens may only contain scopes with the `public` property set to `true`. Secret tokens may contain any scope.

Property | Description
--- | ---
`id` | identifier of the scope
`description` | human friendly description of the scope
`public` | `true` if the scope is available for public tokens

```endpoint
GET /scopes/v1/{username}?access_token={your_access_token} scopes:list
```

#### Example request

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

#### Example response (truncated)

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
