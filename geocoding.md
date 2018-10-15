## 地理编码

Mapbox Geocoding API 提供正向和反向地理编码。

正向地理编码将地理位置文本转换成地理坐标，如将 `2 Lincoln Memorial Circle NW` 转换成 `-77.050,38.889`。

反向地理编码将地理坐标转换成地点名称。如将 `-77.050, 38.889` 转换成 `2 Lincoln Memorial Circle NW`。地点名称可以是具体的地址，也可能是包含给定坐标的地区或国家。

 [MapboxGeocoder.swift](https://github.com/mapbox/MapboxGeocoder.swift) 库提供了对Swift 和 Objective-C语言中地理编码对支持。

**限制条件**

使用地理编码API，您需要一个访问token。每种账号对速率的限制如下所示：

| 账号等级 | 速率限制 |
| ------------- | ---------- |
| 现收现付   | 600 次/分钟 |
| 商业    | 600 次/分钟 |
| 企业    | 2,400 次/分钟 |

超过上面的限制将会返回`HTTP 429` 响应。有关速度限制的信息请查看 [速率限制](#rate-limits)。

批量地理编码仅有[企业计划](https://www.mapbox.com/enterprise/)支持，其他计划类型只支持每次请求单个地理编码。

使用 `mapbox.places` 模式得到的地理编码[必须显示在Mapbox地图中且不能被持久化存储](https://www.mapbox.com/tos/#%5BYmouYmoq%5D)。[企业计划](https://www.mapbox.com/enterprise/) 的 `mapbox.places-permanent` 模式则没有此限制。

[Mapbox 地理编码API覆盖地图](https://www.mapbox.com/geocoding/#coverage) 列出全球范围内所有支持的地理编码结果类型。

查询限制为由空格和标点分隔的20个单词或数字，或者256个字符。

如果使用可选的边界框筛选返回结果，请注意边界框不能跨越子午线。

```objc
@import MapboxGeocoder;

MBGeocoder *geocoder = [[MBGeocoder alloc] initWithAccessToken:@"<#your access token#>"];
```

```objc
@import MapboxGeocoder;

// If you’ve already set `MGLMapboxAccessToken` in your Info.plist:
MBGeocoder *geocoder = [MBGeocoder sharedGeocoder];
```

```swift
import MapboxGeocoder

let geocoder = Geocoder(accessToken: "<#your access token#>")
```

```swift
import MapboxGeocoder

// If you’ve already set `MGLMapboxAccessToken` in your Info.plist:
let geocoder = Geocoder.shared
```

```python
from mapbox import Geocoder
```

```javascript
const mbxGeocoding = require('@mapbox/mapbox-sdk/services/geocoding');
const geocodingClient = mbxGeocoding({ accessToken: '{your_access_token}' });
```

### 请求格式

正向和反向地理编码使用相同的基本格式。由于`{query}` 参数可以包含任何值，所以它需要使用URl编码的UTF-8格式。

```endpoint
GET /geocoding/v5/{mode}/{query}.json
```

URL 参数 | 描述
--- | ---
`query` | 地点。正向地理编码时它是地点名，反向地理编码时它是经纬坐标 (longitude, latitude)。
`mode` | `mapbox.places` 为短暂性地理编码，`mapbox.places-permanent` 可存储编码结果和批量编码。


Query 参数 | 描述
----------|------------
`country` <br /> (可选) | 限制结果为一个或多个国家。 选项为使用逗号分隔的 [ISO 3166 alpha 2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) 国家代码。
`proximity`<br /> (可选) | 基于给定位置的局部结果偏置。选项为`longitude,latitude`坐标。
`types`<br /> (可选) | 根据一个或多个要素过滤结果。选项为 `country`, `region`, `postcode`, `district`, `place`, `locality`, `neighborhood`, `address`, `poi`, 和 `poi.landmark`. `poi.landmark` 返回为 `poi` 类型结果的子集。 多个选项由逗号分隔。
`autocomplete`<br /> (可选) | _仅正向地理编码使用。_ 是否返回自动完成结果。 选项为 `true` 或 `false` ，默认为 `true`.
`bbox`<br /> (可选) | _仅正向地理编码使用。_ 限制结果在边界框之内。选项格式为： `minX,minY,maxX,maxY`.
`limit`<br /> (可选) | 限制返回结果数量。正向地理编码默认为 `5`，反向地理编码默认为  `1`。
`language` <br /> (可选) | 指定响应文本、正向地理编码、查询结果权重所使用的语言。选项为由[ISO 639-1 language code](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) 组成的 [IETF language tags](https://en.wikipedia.org/wiki/IETF_language_tag) 并且可选一个或多个IETF子标签。多个值之间使用逗号分隔。

`{proximity}`参数偏向于搜索给定 `{longitude},{latitude}` 坐标位置5英里范围内的结果。搜索结果不会根据位置或距离排序，但会根据文本相关性和其他条件排序。

`{autocomplete}` 参数控制是否包含自动完成的结果。自动完成可以部分匹配查询条件。例如搜索 `washingto` 结果会包含 `washington` ，即使仅仅匹配了前缀。自动完成对于在用户界面中提供快速的、预键入结果很有用。如果你的查询中使用完整的地址或地方名称，可以通过设置 `{autocomplete}` 为 `false`，从而关闭该行为，执行部分匹配。

`{limit}`参数指定返回结果的最大数量。正向地理编码的默认值为 `5` ，最大值为`10`。
反向地理编码默认值为`1`，最大值为`5`。如果反向地理编码的 `limit` 不是  `1`，则必须指定单一 `types` 。

 `{language}` 参数指定响应所期望使用的语言。对于正向地理编码，与请求语言相匹配的结果比其他语言更受青睐。如果提供了超过一种语言，将会返回所有请求语言的文本。正向地理编码中使用超过一种语言标签时，第一种语言将会用于加权结果中。

任何有效的IETF语言标签都可以提交，最好的情形是返回结果所使用的语言与请求语言相同。当请求语言不可用时会首先回退到相似的语言，其次回退到公用语言。如果使用了回退语言，响应中 `language` 字段值将与请求中的值不再相同。


翻译可用性因语言和地区而异：

**全球覆盖:** _这些语言总是出现在 `国家`, `地区`, 和 显著的 `地方` 要素中。_

语言 | | | 
---|---|---
`de` 德语  | `en` 英语 | `fr` 法语
`it` 意大利语 | `nl` 荷兰语  |

**局部覆盖:** _这些语言可能缺乏全球性的覆盖，但几乎总是出现在“国家”、“地区”和显著的“地方”要素中，并被广泛使用。_

语言 | | | 
--- | --- | ---
`bs` 波斯尼亚语  | `bg` 保加利亚语 | `ca` 加泰罗尼亚语
`cs` 捷克语    | `da` 丹麦语    | `el` 希腊语
`et` 爱沙尼亚语 | `fi` 芬兰语   | `ka` 格鲁吉亚语
`he` 希伯来语   | `hu` 匈牙利语 | `is` 冰岛语
`ja` 日语 | `ko` 韩语    | `lt` 立陶宛语
`lv` 拉脱维亚语  | `mn` 蒙古语 | `nb` 挪威语
`pl` 波兰语   | `pt` 葡萄牙语| `ro` 罗马尼亚语
`ru` 俄语  | `sk` 斯洛伐克语    | `sl` 斯洛维尼亚语
`sr` 塞尔维亚人语  | `sv` 瑞典语   | `tl` 塔加拉族语
`th` 泰语     | `uk` 乌克兰语 | `zh-Hans` 简体中文
`zh-Hant` 繁体中文 | | 


**受限覆盖:** _这些语言有时会出现，但覆盖范围往往不一致或受到地域限制。_

Language | | |
--- | --- | ---
`ar` 阿拉伯语  | `es` 西班牙语 | `fa` 波斯语
`kk` 哈萨克语  | `ms` 马来语   | `sq` 阿尔巴尼亚语
`tr` 土耳其语 | `uz` 乌兹别克语   | `vi` 越南语

### 响应对象

由于Mapbox的地理编码数据不断更新和改进，响应对象中的属性值得不到保证，可能在同一个API版本中发生更改。同一版本的属性可能会增加，但不会被删除。地理编码结果格式为[Carmen GeoJSON](https://github.com/mapbox/carmen/blob/master/carmen-geojson.md)。


属性 | 描述
----------|----------
`type` | `"要素集合"`, [GeoJSON specification](https://tools.ietf.org/html/rfc7946)指定的GeoJSON格式。
`query` | 一个由空格和标点符号分隔的最初查询字符串数组。
`features` | 要素对象数组。
`attribution` | 将Mapbox地理编码API的结果赋予Mapbox的字符串，以及指向Mapbox服务条款和数据源的链接。

**要素对象**

`"features"` 数组中的要素对象拥有一下属性。正向地理编码返回的要素根据 `relevance`排序。逆向地理编码则由与查询坐标重叠度最高的要素到重叠度最低的要素之间，根据索引层级排序。

属性 | 描述
----------|----------
`id` | 字符串类型的要素id以 `{type}.{id}` 形式出现，`{type}` 是 `place_type` 字段中最低层级的要素。要素id的 `{id}` 后缀还不稳定，将来的版本中可能会改变。
`type` | `"Feature"`, [GeoJSON specification](https://tools.ietf.org/html/rfc7946) 标准的 GeoJSON 类型
`place_type` | 描述要素类型的数组。 可选项为 `country`, `region`, `postcode`, `district`, `place`, `locality`, `neighborhood`, `address`, `poi`, 和 `poi.landmark`. 大部分要素只有一种类型，但一旦要素含有多种类型，所有的可用类型必须在数组中列出。（例如, 梵蒂冈是 `country`, `region`, 和 `place`。）
`relevance` | 一个分数值从0（最低相关）到0.99（最高相关），用以表示返回要素与查询的匹配程度。你可以使用 `relevance` 属性来去除不完全匹配查询条件的结果。
`address` <br /> (可选) | 返回地址要素的门牌号。与POI要素的 `address` 属性不同，此属性位于`properties` 对象之外。
`properties` | 描述要素的对象。此属性对象还不稳定，仅仅保证存在 [Carmen GeoJSON](https://github.com/mapbox/carmen/blob/master/carmen-geojson.md) 中的属性。在您的实现中，使用它们之前需要检查该属性是否存在。
`properties.address`<br /> (可选) |  返回的`poi` 要素完整的街道地址。不同于`address` 要素的 `address` 属性，该属性在 `properties` 对象内部。
`properties.category`<br /> (可选) | 逗号分隔的 `poi` 要素类别。
`properties.tel`<br /> (可选) | 表示 `poi` 要素的电话号码的格式化字符串。
`properties.maki`<br /> (可选) | 建议的 [Maki](https://www.mapbox.com/maki-icons/) 图标名，用于基于类别的 `poi` 要素的可视化。 
`properties.landmark`<br /> (可选) | 一个布尔值，指示POI要素是否为地标。地标是引人注意或长期存在的场所，如学校，公园，博物馆和礼拜场所。
`properties.wikidata`<br /> (可选) | 返回要素的 [Wikidata](https://wikidata.org) 标识符。
`properties.short_code`<br /> (可选) | 返回要素的 [ISO 3166-1](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) 国家码和 [ISO 3166-2](https://en.wikipedia.org/wiki/ISO_3166-2) 地区码。
`text` | 表示请求语言中的要素的字符串（如果已指定）。
`place_name` | 表示所请求语言中的要素的字符串（如果已指定）及其完整结果层次结构。
`matching_text`<br /> (可选) | 类似于 `text` 字段的字符串，与查询的匹配程度高于指定语言的结果。例如, 使用英语查询 "Köln, Germany" 可能会返回一个 `text` 值为 "Cologne" 和 `matching_text` 值为 "Köln" 的要素。
`matching_place_name`<br /> (可选) | 类似于 `place_name` 字段的字符串，与查询的匹配程度高于指定语言的结果。 例如, 使用英语查询 "Köln, Germany" 可能会返回一个 `place_name` 值为 "Cologne, Germany" 和 `matching_place_name` 值为 "Köln, North Rhine-Westphalia, Germany" 的要素。
`text_{language}`<br /> (可选) | 类似于 `text` 文本字段的字符串，与请求的语言中的查询匹配。此字段仅在请求多种语言时返回，并且将针对每种请求的语言提供。
`place_name_{language}`<br /> (可选) | 类似于 `place_name` 字段的字符串，与所请求语言的查询匹配。此字段仅在请求多种语言时返回，并且将针对每种请求的语言提供。
`language` <br /> (可选) | [IETF language tag](https://en.wikipedia.org/wiki/IETF_language_tag) 的字符串，表示查询的主要语言。
`language_{language}` <br /> (可选) | 使用 [IETF language tag](https://en.wikipedia.org/wiki/IETF_language_tag)的字符串，代表查询的回退语言。 此字段仅在请求多种语言时返回，并且将针对每种请求的语言提供。
`bbox` | 一个代表边界框的数组，格式为[minX,minY,maxX,maxY]。
`center` | 一个 [longitude,latitude] 形式的数组，代表 `bbox` 的中心点。
`geometry` | 描述要素的空间几何形状的对象。
`geometry.type` | `"Point"`, [GeoJSON specification](https://tools.ietf.org/html/rfc7946) 标准的 GeoJSON 类型。
`geometry.coordinates` | [longitude,latitude] 格式的数组，指定 `bbox` 的中心点。
`geometry.interpolated`<br /> (可选) | 一个布尔值，指示是否沿道路网络插入地址。仅在插入要素时才会显示此字段。
`context` | 表示包含父要素的层次结构的数组。每个父要素可以包括任何上述属性。

```json
{
  "type": "FeatureCollection",
  "query": [
    "los",
    "angeles"
  ],
  "features": [
    {
      "id": "place.9962989141465270",
      "type": "Feature",
      "place_type": [
        "place"
      ],
      "relevance": 0.99,
      "properties": {
        "wikidata": "Q65"
      },
      "text": "Los Angeles",
      "place_name": "Los Angeles, California, United States",
      "bbox": [
        -118.529221009603,
        33.901599990108,
        -118.121099990025,
        34.1612200099034
      ],
      "center": [
        -118.2439,
        34.0544
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          -118.2439,
          34.0544
        ]
      },
      "context": [
        {
          "id": "region.3591",
          "short_code": "US-CA",
          "wikidata": "Q99",
          "text": "California"
        },
        {
          "id": "country.3145",
          "short_code": "us",
          "wikidata": "Q30",
          "text": "United States"
        }
      ]
    },
    {
      "id": "place.10952642230180310",
      "type": "Feature",
      "place_type": [
        "place"
      ],
      "relevance": 0.99,
      "properties": {
        "wikidata": "Q16910"
      },
      "text": "Los Ángeles",
      "place_name": "Los Ángeles, Bío Bío, Chile",
      "bbox": [
        -72.68248,
        -37.663862,
        -72.041277,
        -37.178368
      ],
      "center": [
        -72.35,
        -37.46667
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          -72.35,
          -37.46667
        ]
      },
      "context": [
        {
          "id": "region.3552",
          "short_code": "CL-BI",
          "wikidata": "Q2170",
          "text": "Bío Bío"
        },
        {
          "id": "country.344",
          "short_code": "cl",
          "wikidata": "Q298",
          "text": "Chile"
        }
      ]
    },
    {
      "id": "poi.1611278850983920",
      "type": "Feature",
      "place_type": [
        "poi"
      ],
      "relevance": 0.99,
      "properties": {
        "address": "1 World Way",
        "category": "international airport, airport",
        "tel": "(310) 646-5252",
        "wikidata": "Q8731",
        "landmark": true,
        "maki": "airport"
      },
      "text": "Los Angeles International Airport",
      "place_name": "Los Angeles International Airport, 1 World Way, Los Angeles, California 90045, United States",
      "center": [
        -118.408056,
        33.9425
      ],
      "geometry": {
        "coordinates": [
          -118.408056,
          33.9425
        ],
        "type": "Point"
      },
      "context": [
        {
          "id": "neighborhood.33720",
          "text": "Westchester"
        },
        {
          "id": "postcode.8081932850252730",
          "text": "90045"
        },
        {
          "id": "place.9962989141465270",
          "wikidata": "Q65",
          "text": "Los Angeles"
        },
        {
          "id": "region.3591",
          "short_code": "US-CA",
          "wikidata": "Q99",
          "text": "California"
        },
        {
          "id": "country.3145",
          "short_code": "us",
          "wikidata": "Q30",
          "text": "United States"
        }
      ]
    },
    {
      "id": "neighborhood.2104633",
      "type": "Feature",
      "place_type": [
        "neighborhood"
      ],
      "relevance": 0.99,
      "properties": {},
      "text": "Los Angeles Heights - Keystone",
      "place_name": "Los Angeles Heights - Keystone, San Antonio, Texas 78201, United States",
      "bbox": [
        -98.534942,
        29.453364,
        -98.514652,
        29.485214
      ],
      "center": [
        -98.52,
        29.47
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          -98.52,
          29.47
        ]
      },
      "context": [
        {
          "id": "postcode.7069290925572850",
          "text": "78201"
        },
        {
          "id": "place.7705127234253710",
          "wikidata": "Q975",
          "text": "San Antonio"
        },
        {
          "id": "region.3818",
          "short_code": "US-TX",
          "wikidata": "Q1439",
          "text": "Texas"
        },
        {
          "id": "country.3145",
          "short_code": "us",
          "wikidata": "Q30",
          "text": "United States"
        }
      ]
    },
    {
      "id": "poi.17555628334440500",
      "type": "Feature",
      "place_type": [
        "poi"
      ],
      "relevance": 0.99,
      "properties": {
        "address": "3939 S Figueroa St",
        "category": "stadium, arena",
        "tel": "(213) 747-7111",
        "wikidata": "Q849784",
        "landmark": true,
        "maki": "baseball"
      },
      "text": "Los Angeles Memorial Coliseum",
      "place_name": "Los Angeles Memorial Coliseum, 3939 S Figueroa St, Los Angeles, California 90037, United States",
      "center": [
        -118.287778,
        34.014167
      ],
      "geometry": {
        "coordinates": [
          -118.287778,
          34.014167
        ],
        "type": "Point"
      },
      "context": [
        {
          "id": "neighborhood.293901",
          "text": "South Los Angeles"
        },
        {
          "id": "postcode.15612390715732530",
          "text": "90037"
        },
        {
          "id": "place.9962989141465270",
          "wikidata": "Q65",
          "text": "Los Angeles"
        },
        {
          "id": "region.3591",
          "short_code": "US-CA",
          "wikidata": "Q99",
          "text": "California"
        },
        {
          "id": "country.3145",
          "short_code": "us",
          "wikidata": "Q30",
          "text": "United States"
        }
      ]
    }
  ],
  "attribution": "NOTICE: © 2017 Mapbox and its suppliers. All rights reserved. Use of this data is subject to the Mapbox Terms of Service (https://www.mapbox.com/about/maps/). This response and the information it contains may not be retained."
}
```

### 搜索地点

通常叫做 **正向地理编码**。请求与 `{query}` 文本最匹配的要素数据。响应包含一个或多个按照相关性排序的结果。

[Mapbox.js L.mapbox.geocoderControl](https://www.mapbox.com/mapbox.js/api/v2.3.0/l-mapbox-geocodercontrol/) 和 [mapbox-gl-geocoder](https://github.com/mapbox/mapbox-gl-geocoder)  用户界面使用此API。

[试用此API](https://www.mapbox.com/api-playground/#/forward-geocoding/).


```endpoint
GET /geocoding/v5/{mode}/{query}.json
```

#### 请求示例

```curl
$ curl "https://api.mapbox.com/geocoding/v5/mapbox.places/Los%20Angeles.json?access_token={your_access_token}"
```


```javascript
geocodingClient
  .forwardGeocode({
    query: 'Paris, France',
    limit: 2
  })
  .send()
  .then(response => {
    const match = response.body;
  });

// geocoding with proximity
geocodingClient
  .forwardGeocode({
    query: 'Paris, France',
    proximity: [-95.4431142, 33.6875431]
  })
  .send()
  .then(response => {
    const match = response.body;
  });

// geocoding with countries
geocodingClient
  .forwardGeocode({
    query: 'Paris, France',
    countries: ['fr']
  })
  .send()
  .then(response => {
    const match = response.body;
  });

// geocoding with bounding box
geocodingClient
  .forwardGeocode({
    query: 'Paris, France',
    bbox: [2.14, 48.72, 2.55, 48.96]
  })
  .send()
  .then(response => {
    const match = response.body;
  });
```

```python
geocoder = Geocoder()
response = geocoder.forward("200 queen street")
first = response.geojson()['features'][0]
first['place_name']
# '200 Queen St, Saint John, New Brunswick E2L 2X1, Canada'

# Geocoding with proximity
response = geocoder.forward(
    "200 queen street", lon=-66.05, lat=45.27)
first = response.geojson()['features'][0]
first['place_name']
# '200 Queen St, Saint John, New Brunswick E2L 2X1, Canada'
[int(coord) for coord in first['geometry']['coordinates']]
# [-66, 45]

# Geocoding with a bounding box
response = geocoder.forward(
    "washington", bbox=[-78.338320,38.520792,-77.935454,38.864909])
first = response.geojson()['features'][0]
first['place_name']
# 'Washington, Virginia, United States'
[round(coord, 2) for coord in first['geometry']['coordinates']]
# [-78.16, 38.71]

# Limit results to a certain number
response = geocoder.forward(
    "washington", limit=3)
len(response.geojson()['features'])
# 3

# Filter by country code
response = geocoder.forward("200 queen street", country=['us'])
any(['Canada' in f['place_name'] for f in response.geojson()['features']])
# False

# Filter results to only include points of interest
response = geocoder.forward(
    "new york", types=['poi'])
features = response.geojson()['features']
all([f['id'].startswith('poi') for f in features])
# True
```

```curl
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/2+lincoln+memorial+circle+nw.json?access_token={your_access_token}"

# use the country option to limit
# results to Canada and exclude the
# Lincoln Memorial in America
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/2+lincoln+memorial+circle+nw.json?country=ca&access_token={your_access_token}"

# there are many towns named 'Chester', but adding the
# proximity parameter with local coordinates ensures that the town
# of Chester, New Jersey is in the search results
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/chester.json?proximity=-74.70850,40.78375&access_token={your_access_token}"

# search only for countries named Georgia,
# so that we don't find the state in America
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/georgia.json?types=country&access_token={your_access_token}"

# use the bounding box option to limit results for a
# search for "Starbucks" to those within Washington, D.C.
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/starbucks.json?bbox=-77.083056,38.908611,-76.997778,38.959167&access_token={your_access_token}"

# use the limit option to limit the number of
# results to `2`. Even though there are many possible matches
# for `Washington`, this query will only return 2 results.
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/Washington.json?limit=2&access_token={your_access_token}"
```

```bash
mapbox geocode '2 Lincoln Memorial Circle NW' --bbox=[-78.3284,38.6039,-78.0428,38.7841]
```

```java
// Point used for proximity
Point searchLocationPoint = Point.fromLngLat(2.2869, 48.8590);

MapboxGeocoding client = MapboxGeocoding.builder()
  .accessToken("{your_access_token}")
  .proximity(searchLocationPoint)
  .query("Paris, France")
  .build();   
```

```objc
#if !TARGET_OS_TV
@import Contacts;
#endif

MBForwardGeocodeOptions *options = [[MBForwardGeocodeOptions alloc] initWithQuery:@"200 queen street"];

// To refine the search, you can set various properties on the options object.
options.allowedISOCountryCodes = @[@"CA"];
options.focalLocation = [[CLLocation alloc] initWithLatitude:45.3 longitude:-66.1];
options.allowedScopes = MBPlacemarkScopeAddress | MBPlacemarkScopePointOfInterest;

NSURLSessionDataTask *task = [geocoder geocodeWithOptions:options completionHandler:^(NSArray<MBGeocodedPlacemark *> * _Nullable placemarks, NSString * _Nullable attribution, NSError * _Nullable error) {
    MBPlacemark *placemark = placemarks[0];
    NSLog(@"%@", placemark.name);
        // 200 Queen St
    NSLog(@"%@", placemark.qualifiedName);
        // 200 Queen St, Saint John, New Brunswick E2L 2X1, Canada

    CLLocationCoordinate2D coordinate = placemark.location.coordinate;
    NSLog(@"%f, %f", coordinate.latitude, coordinate.longitude);
        // 45.270093, -66.050985

#if !TARGET_OS_TV
    CNPostalAddressFormatter *formatter = [[CNPostalAddressFormatter alloc] init];
    NSLog(@"%@", [formatter stringFromPostalAddress:placemark.postalAddress]);
        // 200 Queen St
        // Saint John New Brunswick E2L 2X1
        // Canada
#endif
}];
```

```swift
#if !os(tvOS)
    import Contacts
#endif

let options = ForwardGeocodeOptions(query: "200 queen street")

// To refine the search, you can set various properties on the options object.
options.allowedISOCountryCodes = ["CA"]
options.focalLocation = CLLocation(latitude: 45.3, longitude: -66.1)
options.allowedScopes = [.address, .pointOfInterest]

let task = geocoder.geocode(options) { (placemarks, attribution, error) in
    guard let placemark = placemarks?.first else {
        return
    }

    print(placemark.name)
        // 200 Queen St
    print(placemark.qualifiedName)
        // 200 Queen St, Saint John, New Brunswick E2L 2X1, Canada

    let coordinate = placemark.location.coordinate
    print("\(coordinate.latitude), \(coordinate.longitude)")
        // 45.270093, -66.050985

    #if !os(tvOS)
        let formatter = CNPostalAddressFormatter()
        print(formatter.string(from: placemark.postalAddress!))
            // 200 Queen St
            // Saint John New Brunswick E2L 2X1
            // Canada
    #endif
}
```

### 检索附近的地点

通常叫做**逆向地理编码**。请求位于输入 `{longitude},{latitude}` 坐标的要素数据。响应中每种类型至少包含一个结果，除非使用 `limit` 参数。


[试用此API](https://www.mapbox.com/api-playground/#/reverse-geocoding/).

```endpoint
GET /geocoding/v5/{mode}/{longitude},{latitude}.json
```

URL 参数 | 描述
----------|------------
`mode` | `mapbox.places` 为短暂性地理编码，`mapbox.places-permanent` 可存储编码结果和批量编码。

查询参数 | 描述
----------|------------
`country`<br /> (可选) |  限制结果为一个或多个国家。 选项为逗号分隔的[ISO 3166 alpha 2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) 国家代码。
`types`<br /> (可选) |  使用一种或多种类型过滤结果。 可选项为: `country`, `region`, `postcode`, `district`, `place`, `locality`, `neighborhood`, `address`, `poi`, 和 `poi.landmark`。多个选项使用逗号分隔。 `poi.landmark` 返回为 `poi` 类型结果的子集。
`limit`<br /> (可选) |  限制返回结果的最大数量。 默认为`1`，最大值为`5`。使用此选项时必须指定 `types` 。
`language`<br /> (可选) | 指定响应文本的语言。选项为由ISO 639-1 language code 组成的 IETF language tags 并且可选一个或多个IETF子标签。多个值之间使用逗号分隔。
`reverseMode`<br /> (可选) | 设置用来排序附近结果的因素。可选项为 `distance`（默认）和 `score` 。

如果请求的坐标不在指定的 `country` 范围内，不会抛出HTTP错误，而是响应一个空的 `features` 数组。

逆向地理编码中可选的 `reverseMode` 参数控制返回是否考虑显著的要素数据。默认返回最近的要素数据。当设置 `reverseMode=score` 时，查询点1公里内的要素的显著性将于临近性一起纳入考虑范围。当 `limit` 参数设置大于1时，该选项不可用。

#### 请求示例

```curl
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/-73.989,40.733.json?access_token={your_access_token}"

# filter results to only include points of interest
curl "https://api.mapbox.com/geocoding/v5/mapbox.places/-73.989,40.733.json?types=poi&access_token={your_access_token}"
```

```javascript
geocodingClient
  .reverseGeocode({
    query: [-95.4431142, 33.6875431],
    limit: 2
  })
  .send()
  .then(response => {
      // match is a GeoJSON document with geocoding matches
    const match = response.body;
  });
```

```python
response = geocoder.reverse(lon=-73.989, lat=40.733)
features = sorted(response.geojson()['features'], key=lambda x: x['place_name'])
for f in features:
    print('{place_name}: {id}'.format(**f))
# 120 East 13th Street, Manhattan, New York, New York 10003... address...
# Greenwich Village... neighborhood...
# Manhattan... locality...
# New York, New York... postcode...
# New York, New York... place...
# New York... region...
# United States: country...

# Limit results to a specific location type
response = geocoder.reverse(lon=-73.989, lat=40.733, limit=1, types=['country'])
features = response.geojson()['features']
len(features)
# 1
print('{place_name}: {id}'.format(**features[0]))
# United States: country...

# Filter by country code
response = geocoder.forward("200 queen street", country=['us'])
any(['Canada' in f['place_name'] for f in response.geojson()['features']])
# False

# Filter results to only include points of interest
response = geocoder.reverse(
    lon=-73.989, lat=40.733, types=['poi'])
features = response.geojson()['features']
all([f['id'].startswith('poi') for f in features])
# True
```

```bash
mapbox geocode --reverse '[-77.4371, 37.5227]'
```

```java
// the location where you'd like to get info about
Point searchLocationPoint = Point.fromLngLat(2.2869, 48.8590);

MapboxGeocoding client = MapboxGeocoding.builder()
  .accessToken("{your_access_token}")
  .query(searchLocationPoint)
  .geocodingTypes(GeocodingCriteria.TYPE_POI)
  .mode(GeocodingCriteria.MODE_PLACES)
  .build();
```

```objc
MBReverseGeocodeOptions *options = [[MBReverseGeocodeOptions alloc] initWithCoordinate: CLLocationCoordinate2DMake(40.733, -73.989)];
// Or perhaps: [[MBReverseGeocodeOptions alloc] initWithLocation:locationManager.location]

NSURLSessionDataTask *task = [geocoder geocodeWithOptions:options completionHandler:^(NSArray<MBGeocodedPlacemark *> * _Nullable placemarks, NSString * _Nullable attribution, NSError * _Nullable error) {
    MBPlacemark *placemark = placemarks[0];
    NSLog(@"%@", placemark.imageName);
        // telephone
    NSLog(@"%@", [placemark.genres componentsJoinedByString:@", "]);
        // computer, electronic
    NSLog(@"%@", placemark.administrativeRegion.name);
        // New York
    NSLog(@"%@", placemark.administrativeRegion.code);
        // US-NY
    NSLog(@"%@", placemark.place.wikidataItemIdentifier);
        // Q60
}];
```

```swift
let options = ReverseGeocodeOptions(coordinate: CLLocationCoordinate2D(latitude: 40.733, longitude: -73.989))
// Or perhaps: ReverseGeocodeOptions(location: locationManager.location)

let task = geocoder.geocode(options) { (placemarks, attribution, error) in
    guard let placemark = placemarks?.first else {
        return
    }

    print(placemark.imageName ?? "")
        // telephone
    print(placemark.genres?.joined(separator: ", ") ?? "")
        // computer, electronic
    print(placemark.administrativeRegion?.name ?? "")
        // New York
    print(placemark.administrativeRegion?.code ?? "")
        // US-NY
    print(placemark.place?.wikidataItemIdentifier ?? "")
        // Q60
}
```

### 批量请求

_该要素仅适用于 `mapbox.places-permanent` 模式。_

```curl
curl "https://api.mapbox.com/geocoding/v5/mapbox.places-permanent/20001;20009;22209.json?access_token={your_access_token}"
```

批量请求和普通请求的参数相同，但可以包含由`;` 分隔的多个查询条件。每个查询条件需要进行URL编码，但`;` 不需要被编码。

使用 `mapbox.places-permanent` 模式，单次请求最多可以由50个正向或反向地理编码组成。响应为单独请求结果组成的数组。批量请求中的每个查询条件单独计数您账户中的速率限制。

```swift
let options = ForwardBatchGeocodeOptions(queries: ["skyline chili", "gold star chili"])
options.focalLocation = locationManager.location
options.allowedScopes = .pointOfInterest

let task = geocoder.batchGeocode(options) { (placemarksByQuery, attributionsByQuery, error) in
    guard let placemarksByQuery = placemarksByQuery else {
        return
    }

    let nearestSkyline = placemarksByQuery[0][0].location
    let distanceToSkyline = nearestSkyline.distance(from: locationManager.location)
    let nearestGoldStar = placemarksByQuery[1][0].location
    let distanceToGoldStar = nearestGoldStar.distance(from: locationManager.location)

    let distance = LengthFormatter().string(fromMeters: min(distanceToSkyline, distanceToGoldStar))
    print("Found a chili parlor \(distance) away.")
}
```

### POI 类别

<!-- preview -->

POI类别搜索支持正向地理编码请求 `poi` 要素类型。使用 `proximity` 查询参数和POI类别，会根据提供的地点返回兴趣点。例如搜索用户附近的餐馆。支持在响应中返回包含POI类别的`properties.category` 属性。

下面是一些常用的POI类别：

目录 | | | |
--- | --- | --- | --- | ---
bakery | bank | bar | cafe | church
cinema | coffee | concert | fast food | finance
gallery | historic | hotel | landmark | museum
music | park | pizza | restaurant | retail
school | shop | tea | theater | university

当前POI类别可能会更改。未来将提供完整的支持列表。

```endpoint
GET /geocoding/v5/{mode}/{category}.json
```

#### 请求示例

```curl
$ curl "https://api.mapbox.com/geocoding/v5/mapbox.places/coffee.json?&proximity=-77.032,38.912&limit=10&types=poi&access_token={your_access_token}"
```
