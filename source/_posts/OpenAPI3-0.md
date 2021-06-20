---
title: OpenAPI 3.0
date: 2021-06-15 16:31:02
tags: [springboot]
---



OpenAPI3.0的部分译文，来自于[Github相关 项目](https://github.com/fishead/OpenAPI-Specification) ，存储避免丢失。
# 开放API规范


#### 版本 3.0.0

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) [RFC2119](https://tools.ietf.org/html/rfc2119) [RFC8174](https://tools.ietf.org/html/rfc8174) when, and only when, they appear in all capitals, as shown here.

This document is licensed under [The Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0.html).

## 介绍

OpenAPI 规范（OAS），是定义一个标准的、与具体编程语言无关的RESTful API的规范。OpenAPI 规范使得人类和计算机都能在“不接触任何程序源代码和文档、不监控网络通信”的情况下理解一个服务的作用。如果您在定义您的 API 时做的很好，那么使用 API 的人就能非常轻松地理解您提供的 API 并与之交互了。

如果您遵循 OpenAPI 规范来定义您的 API，那么您就可以用文档生成工具来展示您的 API，用代码生成工具来自动生成各种编程语言的服务器端和客户端的代码，用自动测试工具进行测试等等。

## 目录
<!-- TOC depthFrom:1 depthTo:3 withLinks:1 updateOnSave:1 orderedList:0 -->

- [定义](#definitions)
	- [开放API文档](#oasDocument)
	- [路径模板](#pathTemplating)
	- [媒体类型](#mediaTypes)
	- [HTTP 状态码](#httpCodes)
- [规范](#specification)
	- [版本](#versions)
	- [格式](#format)
	- [文档结构](#documentStructure)
	- [数据类型](#dataTypes)
	- [富文本格式](#richText)
	- [相对引用](#relativeReferences)
	- [结构](#schema)
		- [OpenAPI 对象](#oasObject)
		- [Info 对象](#infoObject)
		- [Contact 对象](#contactObject)
		- [License 对象](#licenseObject)
		- [Server 对象](#serverObject)
		- [Server Variable 对象](#serverVariableObject)
		- [Components 对象](#componentsObject)
		- [Paths 对象](#pathsObject)
		- [Path Item 对象](#pathItemObject)
		- [Operation 对象](#operationObject)
		- [External Documentation 对象](#externalDocumentationObject)
		- [Parameter 对象](#parameterObject)
		- [Request Body 对象](#requestBodyObject)
		- [Media Type 对象](#mediaTypeObject)
		- [Encoding 对象](#encodingObject)
		- [Responses 对象](#responsesObject)
		- [Response 对象](#responseObject)
		- [Callback 对象](#callbackObject)
		- [Example 对象](#exampleObject)
		- [Link 对象](#linkObject)
		- [Header 对象](#headerObject)
		- [Tag 对象](#tagObject)
		- [Reference 对象](#referenceObject)
		- [Schema 对象](#schemaObject)
		- [Discriminator 对象](#discriminatorObject)
		- [XML 对象](#xmlObject)
		- [Security Scheme 对象](#securitySchemeObject)
		- [OAuth Flows 对象](#oauthFlowsObject)
		- [OAuth Flow 对象](#oauthFlowObject)
		- [Security Requirement 对象](#securityRequirementObject)
	- [规范扩展](#specificationExtensions)
	- [Security Filtering](#securityFiltering)
- [附录 A: 修订历史](#revisionHistory)


<!-- /TOC -->

## <a name="definitions"></a>术语定义

##### <a name="oasDocument"></a>开放API文档

一（或多）份用来定义或描述一个API的文档。

##### <a name="pathTemplating"></a>路径模板

路径模板指用大括号标记来标记一段URL作为可替换的路径参数。

##### <a name="mediaTypes"></a>媒体类型

媒体类型定义分散于多处。
媒体类型定义应当符合[RFC6838](http://tools.ietf.org/html/rfc6838)。

以下是一些媒体类型定义的示例：

```text
  text/plain; charset=utf-8
  application/json
  application/vnd.github+json
  application/vnd.github.v3+json
  application/vnd.github.v3.raw+json
  application/vnd.github.v3.text+json
  application/vnd.github.v3.html+json
  application/vnd.github.v3.full+json
  application/vnd.github.v3.diff
  application/vnd.github.v3.patch
```

##### <a name="httpCodes"></a>HTTP状态码

HTTP状态码被用来表示一次请求的被执行状态。
[RFC7231](http://tools.ietf.org/html/rfc7231#section-6)定义了有效的状态码，可以在[IANA Status Code Registry](http://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml)找到已经被注册的状态码的列表。

## <a name="specification"></a>规范

### <a name="versions"></a>版本

开放API规范使用符合[语义化版本 2.0.0](http://semver.org/spec/v2.0.0.html)(semver)规范的版本号。

语义化版本的`主版本号`、`次版本号`部分（比如`3.0`）应当被用来标记开放API规范的特性变动。通常 *`.修订号`* 版本被用来表示本文档文档的错误修正而不是特性变动。支持开放API规范3.0的工具应该兼容所有3.0.\*的版本，工具不应当关注修订版本号，比如`3.0.0`和`3.0.1`对它来说应该没有任何区别。

此后开放API规范的相同主版本号下更高次要版本的发布不应当对面向低于此次要版本号开发的工具的造成干扰。因此`3.1.0`版本的规范应当可以在面向`3.0.0`版本规范开发的工具内使用。

任何兼容开放API规范 3.\*.\* 的文档应当包含一个[`openapi`](#oasVersion) 字段用来表明它使用的规范的语义化版本。

### <a name="format"></a>格式

一份遵从开放API规范的文档是一个自包含的JSON对象，可以使用JSON或YAML格式编写。

比如一个字段有一组值，用JSON格式表示为：

```json
{
   "field": [ 1, 2, 3 ]
}
```

规范内的所有字段名都是**小写**。

字段分为两种：固定字段和模式字段。固定字段的字段名是确定的，模式字段的字段名需要符合一定的模式。

如果一个对象里有模式字段，那么在这个对象里的模式字段的名字不能有重复的。

为了保留在 YAML 和 JSON 格式之间转换的能力，推荐使用[1.2](http://www.yaml.org/spec/1.2/spec.html)版本的YAML格式，而且还需要符合以下限制：

- Tags 必须被限制在[JSON Schema ruleset](http://www.yaml.org/spec/1.2/spec.html#id2803231)允许的范围内。
- Keys 必须是[YAML Failsafe schema ruleset](http://yaml.org/spec/1.2/spec.html#id2802346)规范定义的纯字符串。

**注意：** 虽然API文档是使用 YAML 或 JSON 格式书写的，但是API的请求体和响应体或者其他内容可以是任何格式。

### <a name="documentStructure"></a>文档结构

一份 OpenAPI 文档可以是单个文件也可以被拆分为多个文件， 连接的部分由用户自行决定。在后一种情形下，必须如 [JSON Schema](http://json-schema.org) 中定义的那样使用 `$ref` 字段来相互引用。

推荐将根开放API文档命名为`openapi.json` 或 `openapi.yaml`。

### <a name="dataTypes"></a>数据类型

在 OAS 中的原始数据类型是基于 [JSON Schema Specification Wright Draft 00](https://tools.ietf.org/html/draft-wright-json-schema-00#section-4.2) 所支持的类型。注意 `integer` 也作为一个被支持的类型并被定义为不包含小数或指数部分的 JSON 数字。
`null` 不是一个被支持的类型 (查看 [`nullable`](#schemaNullable) 来获得替代解决方案)。
Models 使用 [Schema Object](#schemaObject) 定义，这是一个 JSON Schema Specification Wright Draft 00 的扩展。

<a name="dataTypeFormat"></a>原始类型可以有一个可选的修饰属性：`format`。
OAS 使用多个已知的格式来丰富类型定义。尽管如此，为了文档的需要，`format` 属性被设计为一个 `string` 类型的开放属性值，可以包含任意值。比如 `"email"`, `"uuid"` 等未被此规范定义的格式也可以被使用。没有被定义的 `format` 属性类型遵从 JSON Schema 中的类型定义。无法识别某个 `format` 值的工具应该回退到 `type` 值，就像 `format` 未被指定一样。

被 OAS 定义的格式:

通用名 | [`type`](#dataTypes) | [`format`](#dataTypeFormat) | 备注
----------- | ------ | -------- | --------
integer | `integer` | `int32` | 32 位有符号
long | `integer` | `int64` | 64 位有符号
float | `number` | `float` | |
double | `number` | `double` | |
string | `string` | | |
byte | `string` | `byte` | base64 编码的支付
binary | `string` | `binary` | 任意 8进制序列
boolean | `boolean` | | |
date | `string` | `date` | 定义于 `full-date` - [RFC3339](http://xml2rfc.ietf.org/public/rfc/html/rfc3339.html#anchor14)
dateTime | `string` | `date-time` | 定义于 `date-time` - [RFC3339](http://xml2rfc.ietf.org/public/rfc/html/rfc3339.html#anchor14)
password | `string` | `password` | 告知输入界面不应该明文显示输入信息。

### <a name="richText"></a>富文本格式

整个规范中的 `description` 字段被标记为支持 CommonMark markdown 格式。
OpenAPI 相关的工具在支持 [CommonMark 0.27](http://spec.commonmark.org/0.27/) 中描述的富文本格式方面至少需要支持渲染 markerdown。相关工具为了安全考虑可以选择忽略某些 CommonMark 特性。

### <a name="relativeReferences"></a>URL的相对引用

除非明确指定，所有 URL 类型的属性值都可以是相对地址，就如 [RFC3986](https://tools.ietf.org/html/rfc3986#section-4.2) 中定义的那样以 [`Server Object`](#serverObject) 作为 Base URI。

在 `$ref` 中的相对引用以 [JSON Reference](https://tools.ietf.org/html/draft-pbryan-zyp-json-ref-03) 为依据，以当前文档的 URL 作为 base URI. 同时参考 [Reference Object](#referenceObject)。

### <a name="schema"></a>结构

在接下来的叙述中，如果一个字段没有被明确的标记为 **必选** 或者被描述为 **必须** 或 **应当**，那么可以认为它是一个 **可选** 字段

#### <a name="oasObject"></a>OpenAPI 对象

这是[OpenAPI document](#oasDocument)的根文档对象。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="oasVersion"></a>openapi | `string` | **必选**. 这个字符串必须是[开放API规范版本号](#versions)提到的符合[语义化版本号规范](http://semver.org/spec/v2.0.0.html)的版本号。`openapi`字段应该被工具或者客户端用来解释开放API文档。这个值和API [`info.version`](#infoVersion)字符串没有关联。
<a name="oasInfo"></a>info | [Info 对象](#infoObject) | **必选**。此字段提供API相关的元数据。相关工具可能需要这个字段。
<a name="oasServers"></a>servers | [[Server 对象](#serverObject)] | 这是一个Server对象的数组， 提供到服务器的连接信息。如果没有提供`servers`属性或者是一个空数组，那么默认为是[url](#serverUrl)值为`/`的 [Server 对象](#serverObject) 。
<a name="oasPaths"></a>paths | [Paths 对象](#pathsObject) | **必选**。对所提供的API有效的路径和操作。
<a name="oasComponents"></a>components | [Components 对象](#componentsObject) | 一个包含多种结构的元素。
<a name="oasSecurity"></a>security | [[Security Requirement 对象](#securityRequirementObject)] | 声明API使用的安全机制。The list of values includes alternative security requirement objects that can be used. 认证一个请求时仅允许使用一种安全机制。单独的操作可以覆盖这里的定义。
<a name="oasTags"></a>tags | [[Tag 对象](#tagObject)] | 提供更多元数据的一系列标签，标签的顺序可以被转换工具用来决定API的顺序。不是所有被[Operation 对象](#operationObject)用到的标签都必须被声明。没有被声明的标签可能被工具按自己的逻辑任意整理，每个标签名都应该是唯一的。
<a name="oasExternalDocs"></a>externalDocs | [External Documentation 对象](#externalDocumentationObject) | 附加的文档。这个对象可能会被[规范扩展](#specificationExtensions)扩展。

#### <a name="infoObject"></a>Info 对象

这个对象提供API的元数据。如果客户端需要时可能会用到这些元数据，而且可能会被呈现在编辑工具或者文档生成工具中。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="infoTitle"></a>title | `string` | **必选**. 应用的名称。
<a name="infoDescription"></a>description | `string` | 对应用的简短描述。 [CommonMark syntax](http://spec.commonmark.org/) 可以被用来表示富文本呈现。
<a name="infoTermsOfService"></a>termsOfService | `string` | 指向服务条款的URL地址，必须是URL地址格式。
<a name="infoContact"></a>contact | [Contact Object](#contactObject) | 所开放的API的联系人信息。
<a name="infoLicense"></a>license | [License Object](#licenseObject) | 所开放的API的证书信息。
<a name="infoVersion"></a>version | `string` | **必选**. API文档的版本信息（注意：这个版本和[开放API规范版本](#oasVersion)没有任何关系）。

##### Info 对象示例

```
{
  "title": "Sample Pet Store App",
  "description": "This is a sample server for a pet store.",
  "termsOfService": "http://example.com/terms/",
  "contact": {
    "name": "API Support",
    "url": "http://www.example.com/support",
    "email": "support@example.com"
  },
  "license": {
    "name": "Apache 2.0",
    "url": "http://www.apache.org/licenses/LICENSE-2.0.html"
  },
  "version": "1.0.1"
}
```

```yaml
title: Sample Pet Store App
description: This is a sample server for a pet store.
termsOfService: http://example.com/terms/
contact:
  name: API Support
  url: http://www.example.com/support
  email: support@example.com
license:
  name: Apache 2.0
  url: http://www.apache.org/licenses/LICENSE-2.0.html
version: 1.0.1
```

#### <a name="contactObject"></a>Contact 对象

所公开的API的联系人信息

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="contactName"></a>name | `string` | 人或组织的名称。
<a name="contactUrl"></a>url | `string` | 指向联系人信息的URL地址，必须是URL地址格式。
<a name="contactEmail"></a>email | `string` | 人或组织的email地址，必须是email地址格式。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### Contact 对象示例

```json
{
  "name": "API Support",
  "url": "http://www.example.com/support",
  "email": "support@example.com"
}
```

```yaml
name: API Support
url: http://www.example.com/support
email: support@example.com
```

#### <a name="licenseObject"></a>License 对象

公开API的证书信息。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="licenseName"></a>name | `string` | **必选**. API的证书名。
<a name="licenseUrl"></a>url | `string` | 指向API所使用的证书的URL地址，必须是URL地址格式。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### License 对象示例

```json
{
  "name": "Apache 2.0",
  "url": "http://www.apache.org/licenses/LICENSE-2.0.html"
}
```

```yaml
name: Apache 2.0
url: http://www.apache.org/licenses/LICENSE-2.0.html
```

#### <a name="serverObject"></a>Server 对象

表示一个服务器的对象。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="serverUrl"></a>url | `string` | **必选**. 指向目标主机的URL地址。这个URL地址支持服务器变量而且可能是相对路径，表示主机路径是相对于本文档所在的路径。当一个变量被命名为类似`{`brackets`}`时需要替换此变量。
<a name="serverDescription"></a>description | `string` | 一个可选的字符串，用来描述此URL地址。[CommonMark syntax](http://spec.commonmark.org/)可以被用来呈现富文本格式.
<a name="serverVariables"></a>variables | Map[`string`, [Server Variable Object](#serverVariableObject)] | 一组变量和值的映射，这些值被用来替换服务器URL地址内的模板参数。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### Server 对象示例

单个服务器可以这样描述：

```json
{
  "url": "https://development.gigantic-server.com/v1",
  "description": "Development server"
}
```

```yaml
url: https://development.gigantic-server.com/v1
description: Development server
```

以下内容表示的是有多个服务器时应该如何描述，比如OpenAPI 对象的[`servers`](#oasServers)：

```json
{
  "servers": [
    {
      "url": "https://development.gigantic-server.com/v1",
      "description": "Development server"
    },
    {
      "url": "https://staging.gigantic-server.com/v1",
      "description": "Staging server"
    },
    {
      "url": "https://api.gigantic-server.com/v1",
      "description": "Production server"
    }
  ]
}
```

```yaml
servers:
- url: https://development.gigantic-server.com/v1
  description: Development server
- url: https://staging.gigantic-server.com/v1
  description: Staging server
- url: https://api.gigantic-server.com/v1
  description: Production server
```

以下内容展示了如何使用变量来配置服务器：

```json
{
  "servers": [
    {
      "url": "https://{username}.gigantic-server.com:{port}/{basePath}",
      "description": "The production API server",
      "variables": {
        "username": {
          "default": "demo",
          "description": "this value is assigned by the service provider, in this example `gigantic-server.com`"
        },
        "port": {
          "enum": [
            "8443",
            "443"
          ],
          "default": "8443"
        },
        "basePath": {
          "default": "v2"
        }
      }
    }
  ]
}
```

```yaml
servers:
- url: https://{username}.gigantic-server.com:{port}/{basePath}
  description: The production API server
  variables:
    username:
      # note! no enum here means it is an open value
      default: demo
      description: this value is assigned by the service provider, in this example `gigantic-server.com`
    port:
      enum:
        - '8443'
        - '443'
      default: '8443'
    basePath:
      # open meaning there is the opportunity to use special base paths as assigned by the provider, default is `v2`
      default: v2
```

#### <a name="serverVariableObject"></a>Server Variable 对象

表示可用于服务器URL地址模板变量替换的对象。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="serverVariableEnum"></a>enum | [`string`] | 一组可枚举字符串值，当可替换选项只能设置为固定的某些值时使用。
<a name="serverVariableDefault"></a>default | `string` |  **必选**. 当可替换的值没有被使用者指定时使用的默认值。不像[Schema Object's](#schemaObject)的 `default` ，这个值必须由使用者提供。
<a name="serverVariableDescription"></a>description | `string` | 对服务器变量的可选的描述。[CommonMark syntax](http://spec.commonmark.org/)可以被用来呈现富文本格式.

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

#### <a name="componentsObject"></a>Components 对象

包含开放API规范固定的各种可重用组件。当没有被其他对象引用时，在这里定义定义的组件不会产生任何效果。

##### 固定字段

字段名 | 类型 | 描述
---|:---|---
<a name="componentsSchemas"></a> schemas | Map[`string`, [Schema Object](#schemaObject) \| [Reference Object](#referenceObject)] | 定义可重用的 [Schema 对象](#schemaObject) 的对象。
<a name="componentsResponses"></a> responses | Map[`string`, [Response Object](#responseObject) \| [Reference Object](#referenceObject)] | 定义可重用的 [Response 对象](#responseObject) 的对象。
<a name="componentsParameters"></a> parameters | Map[`string`, [Parameter Object](#parameterObject) \| [Reference Object](#referenceObject)] | 定义可重用的 [Parameter 对象](#parameterObject) 的对象。
<a name="componentsExamples"></a> examples | Map[`string`, [Example Object](#exampleObject) \| [Reference Object](#referenceObject)] | 定义可重用的 [Example 对象](#exampleObject) 的对象。
<a name="componentsRequestBodies"></a> requestBodies | Map[`string`, [Request Body Object](#requestBodyObject) \| [Reference Object](#referenceObject)] | 定义可重用的 [Request Body 对象](#requestBodyObject) 的对象。
<a name="componentsHeaders"></a> headers | Map[`string`, [Header Object](#headerObject) \| [Reference Object](#referenceObject)] | 定义可重用的 [Header 对象](#headerObject) 的对象。
<a name="componentsSecuritySchemes"></a> securitySchemes| Map[`string`, [Security Scheme Object](#securitySchemeObject) \| [Reference Object](#referenceObject)] | 定义可重用的 [Security Scheme 对象](#securitySchemeObject) 的对象。
<a name="componentsLinks"></a> links | Map[`string`, [Link Object](#linkObject) \| [Reference Object](#referenceObject)] | 定义可重用的 [Link 对象](#linkObject) 的对象。
<a name="componentsCallbacks"></a> callbacks | Map[`string`, [Callback Object](#callbackObject) \| [Reference Object](#referenceObject)] | 定义可重用的 [Callback 对象](#callbackObject) 的对象。

这个对象可能会被 [规范扩展](#specificationExtensions) 扩展。

上面定义的所有固定字段的值都是对象，对象包含的key的命名必须满足正则表达式： `^[a-zA-Z0-9\.\-_]+$`。

字段名示例:

```text
User
User_1
User_Name
user-name
my.org.User
```

##### Components 对象示例

```json
"components": {
  "schemas": {
    "Category": {
      "type": "object",
      "properties": {
        "id": {
          "type": "integer",
          "format": "int64"
        },
        "name": {
          "type": "string"
        }
      }
    },
    "Tag": {
      "type": "object",
      "properties": {
        "id": {
          "type": "integer",
          "format": "int64"
        },
        "name": {
          "type": "string"
        }
      }
    }
  },
  "parameters": {
    "skipParam": {
      "name": "skip",
      "in": "query",
      "description": "number of items to skip",
      "required": true,
      "schema": {
        "type": "integer",
        "format": "int32"
      }
    },
    "limitParam": {
      "name": "limit",
      "in": "query",
      "description": "max records to return",
      "required": true,
      "schema" : {
        "type": "integer",
        "format": "int32"
      }
    }
  },
  "responses": {
    "NotFound": {
      "description": "Entity not found."
    },
    "IllegalInput": {
      "description": "Illegal input for operation."
    },
    "GeneralError": {
      "description": "General Error",
      "content": {
        "application/json": {
          "schema": {
            "$ref": "#/components/schemas/GeneralError"
          }
        }
      }
    }
  },
  "securitySchemes": {
    "api_key": {
      "type": "apiKey",
      "name": "api_key",
      "in": "header"
    },
    "petstore_auth": {
      "type": "oauth2",
      "flows": {
        "implicit": {
          "authorizationUrl": "http://example.org/api/oauth/dialog",
          "scopes": {
            "write:pets": "modify pets in your account",
            "read:pets": "read your pets"
          }
        }
      }
    }
  }
}
```

```yaml
components:
  schemas:
    Category:
      type: object
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
    Tag:
      type: object
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
  parameters:
    skipParam:
      name: skip
      in: query
      description: number of items to skip
      required: true
      schema:
        type: integer
        format: int32
    limitParam:
      name: limit
      in: query
      description: max records to return
      required: true
      schema:
        type: integer
        format: int32
  responses:
    NotFound:
      description: Entity not found.
    IllegalInput:
      description: Illegal input for operation.
    GeneralError:
      description: General Error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/GeneralError'
  securitySchemes:
    api_key:
      type: apiKey
      name: api_key
      in: header
    petstore_auth:
      type: oauth2
      flows:
        implicit:
          authorizationUrl: http://example.org/api/oauth/dialog
          scopes:
            write:pets: modify pets in your account
            read:pets: read your pets
```

#### <a name="pathsObject"></a>Paths 对象

定义各个的端点和操作的相对路径。这里指定的路径会和 [`Server 对象`](#serverObject) 内指定的URL地址组成完整的URL地址，路径可以为空，这依赖于 [ACL constraints](#securityFiltering) 的设置。

##### 模式字段

字段名模式 | 类型 | 描述
---|:---:|---
<a name="pathsPath"></a>/{path} | [Path Item 对象](#pathItemObject) | 到各个端点的相对路径，路径必须以`/`打头，这个路径会被**直接连接**到 [`Server 对象`](#serverObject) 的`url`字段以组成完整URL地址（不会考虑是否是相对路径）。这里可以使用 [Path templating](#pathTemplating) ，当做URL地址匹配时，不带路径参数的路径会被优先匹配。应该避免定义多个具有相同路径层级但是路径参数名不同的路径，因为他们是等价的。当匹配出现歧义时，由使用的工具自行决定使用那个路径。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### 路径模板匹配

假设有以下路径，明确定义的路径 `/pets/mine` 会被优先匹配：

```text
  /pets/{petId}
  /pets/mine
```

以下路径被认为是等价的而且是无效的：

```text
  /pets/{petId}
  /pets/{name}
```

以下路径会产生歧义：

```text
  /{entity}/me
  /books/{id}
```

##### Paths 对象示例

```json
{
  "/pets": {
    "get": {
      "description": "Returns all pets from the system that the user has access to",
      "responses": {
        "200": {
          "description": "A list of pets.",
          "content": {
            "application/json": {
              "schema": {
                "type": "array",
                "items": {
                  "$ref": "#/components/schemas/pet"
                }
              }
            }
          }
        }
      }
    }
  }
}
```

```yaml
/pets:
  get:
    description: Returns all pets from the system that the user has access to
    responses:
      '200':
        description: A list of pets.
        content:
          application/json:
            schema:
              type: array
              items:
                $ref: '#/components/schemas/pet'
```

#### <a name="pathItemObject"></a>Path Item 对象

描述对一个路径可执行的有效操作。依赖与 [ACL constraints](#securityFiltering) 的设置，一个Path Item可以是一个空对象，文档的读者仍然可以看到这个路径，但是他们将无法了解到对这个路径可用的任何操作和参数。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="pathItemRef"></a>$ref | `string` | 指定对此路径的外部定义的引用，引用的格式必须符合 [Path Item 对象](#pathItemObject) 的格式，如果引用的外部定义和此对象内的其他定义有冲突，该如何处理冲突尚未被定义。
<a name="pathItemSummary"></a>summary| `string` | 一个可选的简要总结字符串，用来描述此路径内包含的所有操作。
<a name="pathItemDescription"></a>description | `string` | 一个可选的详细说明字符串，用于描述此路径包含的所有操作。 [CommonMark syntax](http://spec.commonmark.org/)可以被用来呈现富文本格式.
<a name="pathItemGet"></a>get | [Operation 对象](#operationObject) | 定义适用于此路径的 GET 操作。
<a name="pathItemPut"></a>put | [Operation 对象](#operationObject) | 定义适用于此路径的 PUT 操作。
<a name="pathItemPost"></a>post | [Operation 对象](#operationObject) | 定义适用于此路径的 POST 操作.
<a name="pathItemDelete"></a>delete | [Operation 对象](#operationObject) | 定义适用于此路径的 DELETE 操作。
<a name="pathItemOptions"></a>options | [Operation 对象](#operationObject) | 定义适用于此路径的 OPTIONS 操作。
<a name="pathItemHead"></a>head | [Operation 对象](#operationObject) | 定义适用于此路径的 HEAD 操作。
<a name="pathItemPatch"></a>patch | [Operation 对象](#operationObject) | 定义适用于此路径的 PATCH 操作。
<a name="pathItemTrace"></a>trace | [Operation 对象](#operationObject) | 定义适用于此路径的 TRACE 操作。
<a name="pathItemServers"></a>servers | [[Server 对象](#serverObject)] | 一个可用于此路径所有操作的替代根`server`的数组定义。
<a name="pathItemParameters"></a>parameters | [[Parameter 对象](#parameterObject) \| [Reference 对象](#referenceObject)] | 一个可用于此路径下所有操作的参数的列表。这些参数可以被具体的操作定义覆盖，但是不能被移除。这个列表禁止包含重复的参数，一个唯一的参数名由 [name](#parameterName) 和 [location](#parameterIn) 的组合来定义。这个列表可以使用 [Reference](#referenceObject) 格式引用定义在 [OpenAPI 对象 components/parameters](#componentsParameters) 内的参数。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### Path Item 对象示例

```json
{
  "get": {
    "description": "Returns pets based on ID",
    "summary": "Find pets by ID",
    "operationId": "getPetsById",
    "responses": {
      "200": {
        "description": "pet response",
        "content": {
          "*/*": {
            "schema": {
              "type": "array",
              "items": {
                "$ref": "#/components/schemas/Pet"
              }
            }
          }
        }
      },
      "default": {
        "description": "error payload",
        "content": {
          "text/html": {
            "schema": {
              "$ref": "#/components/schemas/ErrorModel"
            }
          }
        }
      }
    }
  },
  "parameters": [
    {
      "name": "id",
      "in": "path",
      "description": "ID of pet to use",
      "required": true,
      "schema": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "style": "simple"
    }
  ]
}
```

```yaml
get:
  description: Returns pets based on ID
  summary: Find pets by ID
  operationId: getPetsById
  responses:
    '200':
      description: pet response
      content:
        '*/*' :
          schema:
            type: array
            items:
              $ref: '#/components/schemas/Pet'
    default:
      description: error payload
      content:
        'text/html':
          schema:
            $ref: '#/components/schemas/ErrorModel'
parameters:
- name: id
  in: path
  description: ID of pet to use
  required: true
  schema:
    type: array
    style: simple
    items:
      type: string
```

#### <a name="operationObject"></a>Operation Object

描述对路径的某个操作。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="operationTags"></a>tags | [`string`] | 用于控制API文档的标签列表，标签可以用于在逻辑上分组对资源的操作或作为其它用途的先决条件。
<a name="operationSummary"></a>summary | `string` | 对此操作行为的简短描述。
<a name="operationDescription"></a>description | `string` | 对此操作行为的详细解释。[CommonMark syntax](http://spec.commonmark.org/)可以被用来呈现富文本格式.
<a name="operationExternalDocs"></a>externalDocs | [External Documentation 对象](#externalDocumentationObject) | 附加的外部文档。
<a name="operationId"></a>operationId | `string` | 用于标识此操作的唯一字符串，这个id在此API内包含的所有操作中必须是唯一的。相关的工具和库可能会使用此operationId来唯一的标识一个操作，因此推荐在命名时符合一般的编程命名习惯。
<a name="operationParameters"></a>parameters | [[Parameter 对象](#parameterObject) \| [Reference 对象](#referenceObject)] | 定义可用于此操作的参数列表，如果一个同名的参数已经存在于 [Path Item](#pathItemParameters)，那么这里的定义会覆盖它但是不能移除上面的定义。这个列表不允许包含重复的参数，参数的唯一性由 [name](#parameterName) 和  [location](#parameterIn) 的组合来确定。这个列表可以使用 [Reference 对象](#referenceObject) 来连接定义于 [OpenAPI 对象 components/parameters](#componentsParameters) 的参数。
<a name="operationRequestBody"></a>requestBody | [Request Body 对象](#requestBodyObject) \| [Reference 对象](#referenceObject) | 可用于此操作的请求体。`requestBody` 只能被用于HTTP 1.1 规范 [RFC7231](https://tools.ietf.org/html/rfc7231#section-4.3.1) 中明确定义了包含请求体的请求方法，在其他没有明确定义的请求方法中，`requestBody`的消费者应该应该忽略`requestBody`。
<a name="operationResponses"></a>responses | [Responses 对象](#responsesObject) | **必选**. 定义执行此操作后的可能的响应值列表。
<a name="operationCallbacks"></a>callbacks | Map[`string`, [Callback 对象](#callbackObject) \| [Reference 对象](#referenceObject)] | 一组相对于父操作的可能出现的回调映射，A map of possible out-of band callbacks related to the parent operation. 映射中的每一个键都唯一的映射一个 [Callback 对象](#callbackObject)， that describes a request that may be initiated by the API provider and the expected responses. The key value used to identify the callback object is an expression, evaluated at runtime, that identifies a URL to use for the callback operation.
<a name="operationDeprecated"></a>deprecated | `boolean` | 声明此操作已经被废弃，使用者应该尽量避免使用此操作，默认的值是 `false`。
<a name="operationSecurity"></a>security | [[Security Requirement 对象](#securityRequirementObject)] | 声明那种安全机制可用于此操作。这个列表可以包含多种可用于此操作的安全需求对象，但是在认证一个请求时应该仅使用其中一种。这里的定义会覆盖任何在顶层 [`security`](#oasSecurity) 中的安全声明，因此可以声明一个空数组来变相的移除顶层的安全声明。
<a name="operationServers"></a>servers | [[Server 对象](#serverObject)] | 一个可用于此操作的额外的 `server` 数组，这里的定义会覆盖 Path Item 对象 或 顶层的定义。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### Operation 对象示例

```json
{
  "tags": [
    "pet"
  ],
  "summary": "Updates a pet in the store with form data",
  "operationId": "updatePetWithForm",
  "parameters": [
    {
      "name": "petId",
      "in": "path",
      "description": "ID of pet that needs to be updated",
      "required": true,
      "schema": {
        "type": "string"
      }
    }
  ],
  "requestBody": {
    "content": {
      "application/x-www-form-urlencoded": {
        "schema": {
          "type": "object",
           "properties": {
              "name": {
                "description": "Updated name of the pet",
                "type": "string"
              },
              "status": {
                "description": "Updated status of the pet",
                "type": "string"
             }
           },
        "required": ["status"]
        }
      }
    }
  },
  "responses": {
    "200": {
      "description": "Pet updated.",
      "content": {
        "application/json": {},
        "application/xml": {}
      }
    },
    "405": {
      "description": "Invalid input",
      "content": {
        "application/json": {},
        "application/xml": {}
      }
    }
  },
  "security": [
    {
      "petstore_auth": [
        "write:pets",
        "read:pets"
      ]
    }
  ]
}
```

```yaml
tags:
- pet
summary: Updates a pet in the store with form data
operationId: updatePetWithForm
parameters:
- name: petId
  in: path
  description: ID of pet that needs to be updated
  required: true
  schema:
    type: string
requestBody:
  content:
    'application/x-www-form-urlencoded':
      schema:
       properties:
          name:
            description: Updated name of the pet
            type: string
          status:
            description: Updated status of the pet
            type: string
       required:
         - status
responses:
  '200':
    description: Pet updated.
    content:
      'application/json': {}
      'application/xml': {}
  '405':
    description: Invalid input
    content:
      'application/json': {}
      'application/xml': {}
security:
- petstore_auth:
  - write:pets
  - read:pets
```

#### <a name="externalDocumentationObject"></a>External Documentation 对象

允许引用外部资源来扩展文档。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="externalDocDescription"></a>description | `string` | 对引用的外部文档的简短描述。[CommonMark syntax](http://spec.commonmark.org/)可以被用来呈现富文本格式.
<a name="externalDocUrl"></a>url | `string` | **必选**. 外部文档的URL地址，这个值必须是URL地址格式。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### External Documentation 对象示例

```json
{
  "description": "Find more info here",
  "url": "https://example.com"
}
```

```yaml
description: Find more info here
url: https://example.com
```

#### <a name="parameterObject"></a>Parameter Object

描述一个操作参数。

一个参数的唯一性由 [name](#parameterName) 和 [location](#parameterIn) 的组合来确定。

##### 参数位置

有4种可能的参数位置值可用于`in`字段：

- path - 与 [Path Templating](#pathTemplating) 一起使用，当参数的值是URL操作路径的一部分时可以使用，但是不包含主机地址或基础路径。比如在路径  `/items/{itemId}` 中，路径参数是 `itemId`。
- query - 追加在URL地址之后的参数，比如 `/items?id=###` 中，查询参数是 `id`。
- header - 请求中使用的自定义请求头，注意在 [RFC7230](https://tools.ietf.org/html/rfc7230#page-22) 中规定，请求头的命名是不区分大小写的。
- cookie - 用于传递特定的cookie值。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="parameterName"></a>name | `string` | **必选**. 参数的名称。参数名是*区分大小写*。<ul><li>如果 [`in`](#parameterIn) 的值是 `"path"`，那么 `name` 字段的值必须与其关联的 [Paths 对象](#pathsObject) 内 [path](#pathsPath) 字段的定义相呼应，查看 [Path Templating](#pathTemplating) 了解更多信息。<li>如果 [`in`](#parameterIn) 的值是 `"header"` 而且`name`字段的值是`"Accept"`, `"Content-Type"`或 `"Authorization"`之一，那么此参数定义应该被忽略。<li>除此之外的情况，`name`表示 [`in`](#parameterIn) 属性的名字.</ul>
<a name="parameterIn"></a>in | `string` | **必选**. 参数的位置，可能的值有 "query", "header", "path" 或 "cookie"。
<a name="parameterDescription"></a>description | `string` | 对此参数的简要描述，这里可以包含使用示例。[CommonMark syntax](http://spec.commonmark.org/)可以被用来呈现富文本格式.
<a name="parameterRequired"></a>required | `boolean` | 标明此参数是否是必选参数。如果 [参数位置](#parameterIn) 的值是 `path`，那么这个参数一定是 **必选** 的因此这里的值必须是`true`。其他的则视情况而定。此字段的默认值是`false`。
<a name="parameterDeprecated"></a> deprecated | `boolean` | 标明一个参数是被弃用的而且应该尽快移除对它的使用。
<a name="parameterAllowEmptyValue"></a> allowEmptyValue | `boolean` | 设置是否允许传递空参数，这只在参数值为`query`时有效，默认值是`false`。如果同时指定了[`style`](#parameterStyle)属性且值为`n/a`（无法被序列化）,那么此字段 `allowEmptyValue`应该被忽略。

序列化参数的规则有两种。
对于简单的场景， [`schema`](#parameterSchema) 和 [`style`](#parameterStyle) 可以用于描述参数的结构和语法。

字段名 | 类型 | 描述
---|:---:|---
<a name="parameterStyle"></a>style | `string` | 描述根据参数值类型的不同如何序列化参数。默认值为（基于`in`字段的值）：`query` 对应 `form`；`path` 对应 `simple`; `header` 对应 `simple`; `cookie` 对应 `form`。
<a name="parameterExplode"></a>explode | `boolean` | 当这个值为`true`时，参数值类型为`array`或`object`的参数使用数组内的值或对象的键值对生成带分隔符的参数值。对于其他类型的参数，这个字段没有任何影响。当 [`style`](#parameterStyle) 是 `form`时，这里的默认值是 `true`，对于其他 style 值类型，默认值是`false`。
<a name="parameterAllowReserved"></a>allowReserved | `boolean` | 决定此参数的值是否允许不使用%号编码使用定义于 [RFC3986](https://tools.ietf.org/html/rfc3986#section-2.2)内的保留字符 `:/?#[]@!$&'()*+,;=`。 这个属性仅用于`in`的值是`query`时，此字段的默认值是`false`。
<a name="parameterSchema"></a>schema | [Schema 对象](#schemaObject) \| [Reference 对象](#referenceObject) | 定义适用于此参数的类型结构。
<a name="parameterExample"></a>example | Any | 不同媒体类型的示例，示例应该符合响应的结构的编码属性。各个`example`之间应该是独立的，而且如果一个引用的`schema`也包含一个示例，那么这里定义的示例应该 *覆盖* `schema`包含的示例。为了展现无法被恰当的用 JSON 或 YAML 格式展现的示例时，可以使用经过必要的编码的字符串值。
<a name="parameterExamples"></a>examples | Map[ `string`, [Example 对象](#exampleObject) \| [Reference 对象](#referenceObject)] | 不同媒体类型的示例。每个示例应该包含一个对应于指定编码格式的格式正确的值，这个`examples`映射内包含的对象应该不同于`example`内的值。而且如果一个引用的`schema`也包含一个示例，那么这里定义的示例应该 *覆盖* `schema`包含的示例。

对于更复杂的场景，[`content`](#parameterContent)属性可以定义参数的媒体类型和概要。一个参数必须且只能包含`schema`和`content`属性中的一个。当`example` 或`examples`字段提供了`schema`对象时，示例必须遵照参数的序列化策略。

字段名 | 类型 | 描述
---|:---:|---
<a name="parameterContent"></a>content | Map[`string`, [Media Type Object](#mediaTypeObject)] | 一个定义参数如何呈现的键值对映射。键是媒体类型，值是对应媒体类型的示例数据，此键值对只能包含一组键值对。

##### 样式值

已经定义好了一组`style`类型用于支持常见的通用的简单参数序列化。

`样式` | [`类型`](#dataTypes) |  `in` | 描述
----------- | ------ | -------- | --------
matrix |  `primitive`, `array`, `object` |  `path` | Path 样式的参数，参见 [RFC6570](https://tools.ietf.org/html/rfc6570#section-3.2.7)
label | `primitive`, `array`, `object` |  `path` | Label 样式的参数，参见 [RFC6570](https://tools.ietf.org/html/rfc6570#section-3.2.5)
form |  `primitive`, `array`, `object` |  `query`, `cookie` | Form 样式的参数，参见 [RFC6570](https://tools.ietf.org/html/rfc6570#section-3.2.8). 此选项替换定义于OpenAPI 2.0中`collectionFormat`等于`csv` (当 `explode`值为 false)或`multi` (当 `explode`值为 true)的情况。
simple | `array` | `path`, `header` | Simple 样式的参数，参见 [RFC6570](https://tools.ietf.org/html/rfc6570#section-3.2.2). 此选项替换定义于OpenAPI 2.0 中 `collectionFormat`等于`csv`的情况。
spaceDelimited | `array` | `query` | 空格分隔的数组值。此选项替换定义于OpenAPI 2.0 中 `collectionFormat` equal to `ssv`的情况。
pipeDelimited | `array` | `query` | 管道符`|`的数组值。 此选项替换定义于OpenAPI 2.0 中 `collectionFormat` equal to `pipes`的情况。
deepObject | `object` | `query` | 提供一种简单的方法来表示参数中的嵌套对象值.

##### Style 示例

建设一个参数名为`color`包含如下之一的值：

```text
   string -> "blue"
   array -> ["blue","black","brown"]
   object -> { "R": 100, "G": 200, "B": 150 }
```

下面这个表展示了各个不同类型值之间的例子。

[`style`](#dataTypeFormat) | `explode` | `empty` | `string` | `array` | `object`
----------- | ------ | -------- | -------- | --------|-------
matrix | false | ;color | ;color=blue | ;color=blue,black,brown | ;color=R,100,G,200,B,150
matrix | true | ;color | ;color=blue | ;color=blue;color=black;color=brown | ;R=100;G=200;B=150
label | false | .  | .blue |  .blue.black.brown | .R.100.G.200.B.150
label | true | . | .blue |  .blue.black.brown | .R=100.G=200.B=150
form | false | color= | color=blue | color=blue,black,brown | color=R,100,G,200,B,150
form | true | color= | color=blue | color=blue&color=black&color=brown | R=100&G=200&B=150
simple | false | n/a | blue | blue,black,brown | R,100,G,200,B,150
simple | true | n/a | blue | blue,black,brown | R=100,G=200,B=150
spaceDelimited | false | n/a | n/a | blue%20black%20brown | R%20100%20G%20200%20B%20150
pipeDelimited | false | n/a | n/a | blue\|black\|brown | R\|100\|G\|200|G\|150
deepObject | true | n/a | n/a | n/a | color[R]=100&color[G]=200&color[B]=150

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### Parameter 对象示例

一个值数组，数组元素为64位整数值的请求头参数：

```json
{
  "name": "token",
  "in": "header",
  "description": "token to be passed as a header",
  "required": true,
  "schema": {
    "type": "array",
    "items": {
      "type": "integer",
      "format": "int64"
    }
  },
  "style": "simple"
}
```

```yaml
name: token
in: header
description: token to be passed as a header
required: true
schema:
  type: array
  items:
    type: integer
    format: int64
style: simple
```

一个值类型为字符串的路径参数：

```json
{
  "name": "username",
  "in": "path",
  "description": "username to fetch",
  "required": true,
  "schema": {
    "type": "string"
  }
}
```

```yaml
name: username
in: path
description: username to fetch
required: true
schema:
  type: string
```

一个值类型为字符串的可选查询参数，允许通过通过重复参数来传递多个值：

```json
{
  "name": "id",
  "in": "query",
  "description": "ID of the object to fetch",
  "required": false,
  "schema": {
    "type": "array",
    "items": {
      "type": "string"
    }
  },
  "style": "form",
  "explode": true
}
```

```yaml
name: id
in: query
description: ID of the object to fetch
required: false
schema:
  type: array
  items:
    type: string
style: form
explode: true
```

一个任意格式的查询参数，允许使用指定类型的未定义参数：

```json
{
  "in": "query",
  "name": "freeForm",
  "schema": {
    "type": "object",
    "additionalProperties": {
      "type": "integer"
    },
  },
  "style": "form"
}
```

```yaml
in: query
name: freeForm
schema:
  type: object
  additionalProperties:
    type: integer
style: form
```

使用`content`定义序列化方法的复杂参数：

```json
{
  "in": "query",
  "name": "coordinates",
  "content": {
    "application/json": {
      "schema": {
        "type": "object",
        "required": [
          "lat",
          "long"
        ],
        "properties": {
          "lat": {
            "type": "number"
          },
          "long": {
            "type": "number"
          }
        }
      }
    }
  }
}
```

```yaml
in: query
name: coordinates
content:
  application/json:
    schema:
      type: object
      required:
        - lat
        - long
      properties:
        lat:
          type: number
        long:
          type: number
```

#### <a name="requestBodyObject"></a>Request Body Object

定义请求体。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="requestBodyDescription"></a>description | `string` | 对请求体的简要描述，可以包含使用示例，[CommonMark syntax](http://spec.commonmark.org/)可以被用来呈现富文本格式.
<a name="requestBodyContent"></a>content | Map[`string`, [Media Type Object](#mediaTypeObject)] | **必选**. 请求体的内容。请求体的属性key是一个媒体类型或者[媒体类型范围](https://tools.ietf.org/html/rfc7231#appendix-D)，值是对应媒体类型的示例数据。对于能匹配多个key的请求，定义更明确的请求会更优先被匹配。比如`text/plain`会覆盖`text/*`的定义。
<a name="requestBodyRequired"></a>required | `boolean` | 指定请求体是不是应该被包含在请求中，默认值是`false`。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### Request Body 示例

一个引用了模型定义的请求体。

```json
{
  "description": "user to add to the system",
  "content": {
    "application/json": {
      "schema": {
        "$ref": "#/components/schemas/User"
      },
      "examples": {
          "user" : {
            "summary": "User Example",
            "externalValue": "http://foo.bar/examples/user-example.json"
          }
        }
    },
    "application/xml": {
      "schema": {
        "$ref": "#/components/schemas/User"
      },
      "examples": {
          "user" : {
            "summary": "User example in XML",
            "externalValue": "http://foo.bar/examples/user-example.xml"
          }
        }
    },
    "text/plain": {
      "examples": {
        "user" : {
            "summary": "User example in Plain text",
            "externalValue": "http://foo.bar/examples/user-example.txt"
        }
      }
    },
    "*/*": {
      "examples": {
        "user" : {
            "summary": "User example in other format",
            "externalValue": "http://foo.bar/examples/user-example.whatever"
        }
      }
    }
  }
}
```

```yaml
description: user to add to the system
content:
  'application/json':
    schema:
      $ref: '#/components/schemas/User'
    examples:
      user:
        summary: User Example
        externalValue: 'http://foo.bar/examples/user-example.json'
  'application/xml':
    schema:
      $ref: '#/components/schemas/User'
    examples:
      user:
        summary: User Example in XML
        externalValue: 'http://foo.bar/examples/user-example.xml'
  'text/plain':
    examples:
      user:
        summary: User example in text plain format
        externalValue: 'http://foo.bar/examples/user-example.txt'
  '*/*':
    examples:
      user:
        summary: User example in other format
        externalValue: 'http://foo.bar/examples/user-example.whatever'
```

请求体是一个字符串的数组：

```json
{
  "description": "user to add to the system",
  "content": {
    "text/plain": {
      "schema": {
        "type": "array",
        "items": {
          "type": "string"
        }
      }
    }
  }
}
```

```yaml
description: user to add to the system
required: true
content:
  text/plain:
    schema:
      type: array
      items:
        type: string
```

#### <a name="mediaTypeObject"></a>Media Type 对象

每种媒体类型对象都有相应的结构和示例来描述它。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="mediaTypeSchema"></a>schema | [Schema 对象](#schemaObject) \| [Reference 对象](#referenceObject) | 定义此媒体类型的结构。
<a name="mediaTypeExample"></a>example | Any | 媒体类型的示例。示例对象应该符合此媒体类型的格式， 这里指定的`example`对象 object is mutually exclusive of the `examples` object.  而且如果引用的`schema`也包含示例，在这里指定的`example`值将会覆盖`schema`提供的示例。
<a name="mediaTypeExamples"></a>examples | Map[ `string`, [Example 对象](#exampleObject) \| [Reference 对象](#referenceObject)] | 媒体类型的示例，每个媒体对象的值都应该匹配它对应的媒体类型的格式。  The `examples` object is mutually exclusive of the `example` object.  而且如果引用的`schema`也包含示例，在这里指定的`example`值将会覆盖`schema`提供的示例。
<a name="mediaTypeEncoding"></a>encoding | Map[`string`, [Encoding 对象](#encodingObject)] | 属性名与编码信息的映射。每个属性名必须存在于`schema`属性的key中，当媒体类型等于`multipart`或`application/x-www-form-urlencoded`时，编码对象信息仅适用于`requestBody`。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### Media Type 示例

```js
{
  "application/json": {
    "schema": {
         "$ref": "#/components/schemas/Pet"
    },
    "examples": {
      "cat" : {
        "summary": "An example of a cat",
        "value":
          {
            "name": "Fluffy",
            "petType": "Cat",
            "color": "White",
            "gender": "male",
            "breed": "Persian"
          }
      },
      "dog": {
        "summary": "An example of a dog with a cat's name",
        "value" :  {
          "name": "Puma",
          "petType": "Dog",
          "color": "Black",
          "gender": "Female",
          "breed": "Mixed"
        },
      "frog": {
          "$ref": "#/components/examples/frog-example"
        }
      }
    }
  }
}
```

```yaml
application/json:
  schema:
    $ref: "#/components/schemas/Pet"
  examples:
    cat:
      summary: An example of a cat
      value:
        name: Fluffy
        petType: Cat
        color: White
        gender: male
        breed: Persian
    dog:
      summary: An example of a dog with a cat's name
      value:
        name: Puma
        petType: Dog
        color: Black
        gender: Female
        breed: Mixed
    frog:
      $ref: "#/components/examples/frog-example"
```

##### 对文件上传的考虑

相对于2.0的规范，`file`内容的上传与下载在开放API规范与其他类型一样使用相同的语法来描述。
特别的是:

```yaml
# content transferred with base64 encoding
schema:
  type: string
  format: base64
```

```yaml
# content transferred in binary (octet-stream):
schema:
  type: string
  format: binary
```

这些示例同时适用于文件上传和下载。

一个使用`POST`操作提交文件的`requestBody`看起来像下面这样：

```yaml
requestBody:
  content:
    application/octet-stream:
      # any media type is accepted, functionally equivalent to `*/*`
      schema:
        # a binary file of any type
        type: string
        format: binary
```

此外，可以指定明确的媒体类型：

```yaml
# multiple, specific media types may be specified:
requestBody:
  content:
      # a binary file of type png or jpeg
    'image/jpeg':
      schema:
        type: string
        format: binary
    'image/png':
      schema:
        type: string
        format: binary
```

为了同时上传多个文件，必须指定`multipart`媒体类型：

```yaml
requestBody:
  content:
    multipart/form-data:
      schema:
        properties:
          # The property name 'file' will be used for all files.
          file:
            type: array
            items:
              type: string
              format: binary

```

##### x-www-form-urlencoded 请求体的支持

可以使用下面定义的格式来提交form url编码[RFC1866](https://tools.ietf.org/html/rfc1866)的内容：

```yaml
requestBody:
  content:
    application/x-www-form-urlencoded:
      schema:
        type: object
        properties:
          id:
            type: string
            format: uuid
          address:
            # complex types are stringified to support RFC 1866
            type: object
            properties: {}
```

在这个示例中，在内容被传送到服务器之前，`requestBody`中的内容必须使用[RFC1866](https://tools.ietf.org/html/rfc1866/)中定义的方式字符串化。此外`address`字段的复杂对象将会被字符串化。

当使用`application/x-www-form-urlencoded`格式传送复杂对象时，默认的序列化策略在[`Encoding Object`](#encodingObject)的[`style`](#encodingStyle) 属性中定义为`form`.

##### 对`multipart`内容的特别思考

使用`multipart/form-data`作为`Content-Type`来传送请求体是很常见的做法。相对于2.0版本的规范，当定义`multipart`内容的输入参数时必须指定`schema`属性。这不但支持复杂的结构而且支持多文件上传机制。

当使用`multipart`类型是，可以使用boundaries来分隔传送的内容，因此`multipart`定义了以下默认的`Content-Type`：

- 如果属性是一个原始值或者是一个原始值的数组，那么默认的Content-Type是 `text/plain`
- 如果属性是复杂对象或者复杂对象的数组，那么默认的Content-Type是`application/json`
- 如果属性是`type: string`与`format: binary`或`format: base64`(也就是文件对象)的组合，那么默认的Content-Type是 `application/octet-stream`

示例:

```yaml
requestBody:
  content:
    multipart/form-data:
      schema:
        type: object
        properties:
          id:
            type: string
            format: uuid
          address:
            # default Content-Type for objects is `application/json`
            type: object
            properties: {}
          profileImage:
            # default Content-Type for string/binary is `application/octet-stream`
            type: string
            format: binary
          children:
            # default Content-Type for arrays is based on the `inner` type (text/plain here)
            type: array
            items:
              type: string
          addresses:
            # default Content-Type for arrays is based on the `inner` type (object shown, so `application/json` in this example)
            type: array
            items:
              type: '#/components/schemas/Address'
```

这里介绍一下用来控制序列化`multipart`请求体的`encoding`属性，这个属性只适用于`multipart`和`application/x-www-form-urlencoded`类型的请求体。

#### <a name="encodingObject"></a>Encoding 对象

一个编码定义仅适用于一个结构属性。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="encodingContentType"></a>contentType | `string` | 对具体属性的 Content-Type的编码。默认值取决于属性的类型：`application/octet-stream`编码适用于`binary`格式的`string`；`text/plain`适用于其他原始值；`application/json`适用于`object`；对于`array`值类型的默认值取决于数组内元素的类型，默认值可以是明确的媒体类型(比如`application/json`), 或者通配符类型的媒体类型(比如`image/*`), 又或者是用分号分隔的两种媒体类型。
<a name="encodingHeaders"></a>headers | Map[`string`, [Header 对象](#headerObject) \| [Reference 对象](#referenceObject)] | 提供附加信息的请求头键值对映射。比如`Content-Disposition`、`Content-Type`各自描述了不同的信息而且在这里将会被忽略，如果请求体的媒体类型不是`multipart`，这个属性将会被忽略。
<a name="encodingStyle"></a>style | `string` | 描述一个属性根据它的类型将会被如何序列化。查看[Parameter 对象](#parameterObject)的[`style`](#parameterStyle)属性可以得到更多详细信息。这个属性的行为与`query`参数相同，包括默认值的定义。如果请求体的媒体类型不是`application/x-www-form-urlencoded`，这个属性将会被忽略。
<a name="encodingExplode"></a>explode | `boolean` | 当这个值为true时，类型为`array`或`object`的属性值会为数组的每个元素或对象的每个键值对分开生成参数。这个属性对其他数据类型没有影响。当[`style`](#encodingStyle)为`form`时，这个属性的默认值是`true`，对于其他的`style`类型，这个属性的默认值是`false`。这个属性会被忽略如果请求体的媒体类型不是`application/x-www-form-urlencoded`。
<a name="encodingAllowReserved"></a>allowReserved | `boolean` | 决定此参数的值是否允许不使用%号编码使用定义于 [RFC3986](https://tools.ietf.org/html/rfc3986#section-2.2)内的保留字符 `:/?#[]@!$&'()*+,;=`。 这个属性仅用于`in`的值是`query`时，此字段的默认值是`false`。 这个属性会被忽略如果请求体的媒体类型不是`application/x-www-form-urlencoded`。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### Encoding 对象示例

```yaml
requestBody:
  content:
    multipart/mixed:
      schema:
        type: object
        properties:
          id:
            # default is text/plain
            type: string
            format: uuid
          address:
            # default is application/json
            type: object
            properties: {}
          historyMetadata:
            # need to declare XML format!
            description: metadata in XML format
            type: object
            properties: {}
          profileImage:
            # default is application/octet-stream, need to declare an image type only!
            type: string
            format: binary
      encoding:
        historyMetadata:
          # require XML Content-Type in utf-8 encoding
          contentType: application/xml; charset=utf-8
        profileImage:
          # only accept png/jpeg
          contentType: image/png, image/jpeg
          headers:
            X-Rate-Limit-Limit:
              description: The number of allowed requests in the current period
              schema:
                type: integer
```

#### <a name="responsesObject"></a>Responses 对象

描述一个操作可能发生的响应的响应码与响应包含的响应体的对象。

一份API文档不必包含所有可能响应码，因为有些状态码无法提前预知。尽管如此，一份文档还是应当包含所有成功的响应和任何已知的错误响应。

`default`字段可以用来标记一个响应适用于其他未被规范明确定义的HTTP响应码的默认响应。

一个`Responses 对象`必须至少包含一个响应码，而且是成功的响应。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="responsesDefault"></a>default | [Response 对象](#responseObject) \| [Reference 对象](#referenceObject) | 用于描述未被明确声明的HTTP响应码的响应的文档。使用这个字段来覆盖未声明的响应。一个 [Reference 对象](#referenceObject) 可以链接定义于 [OpenAPI 对象 components/responses](#componentsResponses) 区域的响应对象。

##### 模式字段

字段名模式 | 类型 | 描述
---|:---:|---
<a name="responsesCode"></a>[HTTP Status Code](#httpCodes) | [Response 对象](#responseObject) \| [Reference 对象](#referenceObject) | 任何 [HTTP status code](#httpCodes) 都可以被用作属性名， 但是每一个状态码只能使用一次，用于描述此状态码的响应。一个 [Reference 对象](#referenceObject) 可以链接定义于 [OpenAPI 对象 components/responses](#componentsResponses) 区域的响应对象。这个字段名必须包含在双引号中 (例如 "200") 以兼容 JSON 和 YAML。这个字段可以包含大写的通配字符`X`来定义响应码的范围。例如，`2XX` 代表所有位于 `[200-299]` 范围内的响应码。只允许使用以下范围定义：`1XX`, `2XX`, `3XX`, `4XX`, 和 `5XX`。如果同时包含范围定义与明确定义的响应，那么明确定义的响应有更高的优先级。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### Responses 对象示例

一个代表成功操作的 200 响应和一个代表其他操作状态的默认响应（暗示是一个错误）：

```json
{
  "200": {
    "description": "a pet to be returned",
    "content": {
      "application/json": {
        "schema": {
          "$ref": "#/components/schemas/Pet"
        }
      }
    }
  },
  "default": {
    "description": "Unexpected error",
    "content": {
      "application/json": {
        "schema": {
          "$ref": "#/components/schemas/ErrorModel"
        }
      }
    }
  }
}
```

```yaml
'200':
  description: a pet to be returned
  content:
    application/json:
      schema:
        $ref: '#/components/schemas/Pet'
default:
  description: Unexpected error
  content:
    application/json:
      schema:
        $ref: '#/components/schemas/ErrorModel'
```

#### <a name="responseObject"></a>Response对象

描述单个API操作的响应，包括设计时间、基于不同响应也包括到相应操作的静态`links`

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="responseDescription"></a>description | `string` | **必选**. 对响应的简短描述。[CommonMark syntax](http://spec.commonmark.org/)可以被用来呈现富文本格式.
<a name="responseHeaders"></a>headers | Map[`string`, [Header Object](#headerObject)  \| [Reference Object](#referenceObject)] |  映射HTTP头名称到其定义。[RFC7230](https://tools.ietf.org/html/rfc7230#page-22) 规定了HTTP头名称不区分大小写。如果一个响应头使用`"Content-Type"`作为HTTP头名称，它会被忽略。
<a name="responseContent"></a>content | Map[`string`, [Media Type Object](#mediaTypeObject)] | 一个包含描述预期响应负载的映射。使用 media type 或 [media type range](https://tools.ietf.org/html/rfc7231#appendix-D) 作为键，以响应的描述作为值。当一个响应匹配多个键时，只有最明确的键才适用。比如：text/plain 会覆盖 text/*
<a name="responseLinks"></a>links | Map[`string`, [Link Object](#linkObject) \| [Reference Object](#referenceObject)] | A map of operations links that can be followed from the response. The key of the map is a short name for the link, following the naming constraints of the names for [Component Objects](#componentsObject).

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### Response 对象示例s

一个包含复杂类型的数组格式的响应：

```json
{
  "description": "A complex object array response",
  "content": {
    "application/json": {
      "schema": {
        "type": "array",
        "items": {
          "$ref": "#/components/schemas/VeryComplexType"
        }
      }
    }
  }
}
```

```yaml
description: A complex object array response
content:
  application/json:
    schema:
      type: array
      items:
        $ref: '#/components/schemas/VeryComplexType'
```

字符串类型的响应：

```json
{
  "description": "A simple string response",
  "content": {
    "text/plain": {
      "schema": {
        "type": "string"
      }
    }
  }

}
```

```yaml
description: A simple string response
representations:
  text/plain:
    schema:
      type: string
```

带HTTP头的普通文本类型的响应：

```json
{
  "description": "A simple string response",
  "content": {
    "text/plain": {
      "schema": {
        "type": "string"
      }
    }
  },
  "headers": {
    "X-Rate-Limit-Limit": {
      "description": "The number of allowed requests in the current period",
      "schema": {
        "type": "integer"
      }
    },
    "X-Rate-Limit-Remaining": {
      "description": "The number of remaining requests in the current period",
      "schema": {
        "type": "integer"
      }
    },
    "X-Rate-Limit-Reset": {
      "description": "The number of seconds left in the current period",
      "schema": {
        "type": "integer"
      }
    }
  }
}
```

```yaml
description: A simple string response
content:
  text/plain:
    schema:
      type: string
    example: 'whoa!'
headers:
  X-Rate-Limit-Limit:
    description: The number of allowed requests in the current period
    schema:
      type: integer
  X-Rate-Limit-Remaining:
    description: The number of remaining requests in the current period
    schema:
      type: integer
  X-Rate-Limit-Reset:
    description: The number of seconds left in the current period
    schema:
      type: integer
```

没有返回值的响应：

```json
{
  "description": "object created"
}
```

```yaml
description: object created
```

#### <a name="callbackObject"></a>Callback 对象

A map of possible out-of band callbacks related to the parent operation.
映射中的每个值都是一个描述一组可能会被API提供者发起的请求和相应的响应的 [Path Item Object](#pathItemObject) 。用以标识回调对象的键是一个表达式，表达式会在运行时被计算，得到的值作为回调操作的URL。

##### 模式字段

字段名模式 | 类型 | 描述
---|:---:|---
<a name="callbackExpression"></a>{expression} | [Path Item Object](#pathItemObject) | 一个用于定义回调请求和响应的 Path Item Object。 A [complete example](../examples/v3.0/callback-example.yaml) is available.

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### Key Expression

用于标识 [Path Item Object](#pathItemObject) 的键是一个 [runtime expression](#runtimeExpression)，此表达式会在运行时的HTTP请求/响应上下文中被计算，计算结果用于表示回调请求的URL。
一个简单的例子是 `$request.body#/url`。
However, using a [runtime expression](#runtimeExpression) the complete HTTP message can be accessed.
This includes accessing any part of a body that a JSON Pointer [RFC6901](https://tools.ietf.org/html/rfc6901) can reference.

比如有如下 HTTP 请求：

```http
POST /subscribe/myevent?queryUrl=http://clientdomain.com/stillrunning HTTP/1.1
Host: example.org
Content-Type: application/json
Content-Length: 187

{
  "failedUrl" : "http://clientdomain.com/failed",
  "successUrls" : [
    "http://clientdomain.com/fast",
    "http://clientdomain.com/medium",
    "http://clientdomain.com/slow"
  ]
}

201 Created
Location: http://example.org/subscription/1
```

下方示例展示了各种表达式是如何被计算，这里假设回调操作有一个名为 `eventType` 的路径参数和一个名为 `queryUrl` 的查询参数。

Expression | Value
---|:---
$url | [http://example.org/subscribe/myevent?queryUrl=http://clientdomain.com/stillrunning]
$method | POST
$request.path.eventType | myevent
$request.query.queryUrl | [http://clientdomain.com/stillrunning]
$request.header.content-Type | application/json
$request.body#/failedUrl | [http://clientdomain.com/stillrunning]
$request.body#/successUrls/2 | [http://clientdomain.com/medium]
$response.header.Location | [http://example.org/subscription/1]

##### Callback 对象示例

如下示例展示了一个通过请求体内的 `id` 和 `email` 属性指定的URL的回调。

```yaml
myWebhook:
  'http://notificationServer.com?transactionId={$request.body#/id}&email={$request.body#/email}':
    post:
      requestBody:
        description: Callback payload
        content:
          'application/json':
            schema:
              $ref: '#/components/schemas/SomePayload'
      responses:
        '200':
          description: webhook successfully processed and no retries will be performed
```

#### <a name="exampleObject"></a>Example 对象

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="exampleSummary"></a>summary | `string` | example 的简要描述。
<a name="exampleDescription"></a>description | `string` | example 的详细描述。[CommonMark syntax](http://spec.commonmark.org/)可以被用来呈现富文本格式.
<a name="exampleValue"></a>value | Any | 嵌入的字面量 example。 `value`  字段和 `externalValue` 字段是互斥的。无法使用 JSON 或 YAML 表示的媒体类型可以使用字符串值来表示。
<a name="exampleExternalValue"></a>externalValue | `string` | 指向字面 exmaple 的一个 URL。这提供了引用无法被包含在 JSON 或 YAML 文档中的 example。`value`  字段和 `externalValue` 字段是互斥的。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

In all cases, the example value is expected to be compatible with the type schema
of its associated value.  Tooling implementations MAY choose to
validate compatibility automatically, and reject the example value(s) if incompatible.

##### Example 对象示例

```yaml
# in a model
schemas:
  properties:
    name:
      type: string
      examples:
        name:
          $ref: http://example.org/petapi-examples/openapi.json#/components/examples/name-example

# in a request body:
  requestBody:
    content:
      'application/json':
        schema:
          $ref: '#/components/schemas/Address'
        examples:
          foo:
            summary: A foo example
            value: {"foo": "bar"}
          bar:
            summary: A bar example
            value: {"bar": "baz"}
      'application/xml':
        examples:
          xmlExample:
            summary: This is an example in XML
            externalValue: 'http://example.org/examples/address-example.xml'
      'text/plain':
        examples:
          textExample:
            summary: This is a text example
            externalValue: 'http://foo.bar/examples/address-example.txt'


# in a parameter
  parameters:
    - name: 'zipCode'
      in: 'query'
      schema:
        type: 'string'
        format: 'zip-code'
        examples:
          zip-example:
            $ref: '#/components/examples/zip-example'

# in a response
  responses:
    '200':
      description: your car appointment has been booked
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/SuccessResponse'
          examples:
            confirmation-success:
              $ref: '#/components/examples/confirmation-success'
```

#### <a name="linkObject"></a>Link 对象

The `Link 对象` represents a possible design-time link for a response.
The presence of a link does not guarantee the caller's ability to successfully invoke it, rather it provides a known relationship and traversal mechanism between responses and other operations.

Unlike _dynamic_ links (i.e. links provided **in** the response payload), the OAS linking mechanism does not require link information in the runtime response.

For computing links, and providing instructions to execute them, a [runtime expression](#runtimeExpression) is used for accessing values in an operation and using them as parameters while invoking the linked operation.

##### 固定字段

字段名  |  Type  | 描述
---|:---:|---
<a name="linkOperationRef"></a>operationRef | `string` | A relative or absolute reference to an OAS operation. This field is mutually exclusive of the `operationId` field, and MUST point to an [Operation Object](#operationObject). Relative `operationRef` values MAY be used to locate an existing [Operation Object](#operationObject) in the OpenAPI definition.
<a name="linkOperationId"></a>operationId  | `string` | The name of an _existing_, resolvable OAS operation, as defined with a unique `operationId`.  This field is mutually exclusive of the `operationRef` field.
<a name="linkParameters"></a>parameters   | Map[`string`, Any \| [{expression}](#runtimeExpression)] | A map representing parameters to pass to an operation as specified with `operationId` or identified via `operationRef`. The key is the parameter name to be used, whereas the value can be a constant or an expression to be evaluated and passed to the linked operation.  The parameter name can be qualified using the [parameter location](#parameterIn) `[{in}.]{name}` for operations that use the same parameter name in different locations (e.g. path.id).
<a name="linkRequestBody"></a>requestBody | Any \| [{expression}](#runtimeExpression) | A literal value or [{expression}](#runtimeExpression) to use as a request body when calling the target operation.
<a name="linkDescription"></a>description  | `string` | A description of the link. [CommonMark syntax](http://spec.commonmark.org/)可以被用来呈现富文本格式.
<a name="linkServer"></a>server       | [Server Object](#serverObject) | A server object to be used by the target operation.

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

A linked operation MUST be identified using either an `operationRef` or `operationId`.
In the case of an `operationId`, it MUST be unique and resolved in the scope of the OAS document.
Because of the potential for name clashes, the `operationRef` syntax is preferred
for specifications with external references.

##### Examples

Computing a link from a request operation where the `$request.path.id` is used to pass a request parameter to the linked operation.

```yaml
paths:
  /users/{id}:
    parameters:
    - name: id
      in: path
      required: true
      description: the user identifier, as userId
      schema:
        type: string
    get:
      responses:
        '200':
          description: the user being returned
          content:
            application/json:
              schema:
                type: object
                properties:
                  uuid: # the unique user id
                    type: string
                    format: uuid
        links:
          address:
            # the target link operationId
            operationId: getUserAddress
            parameters:
              # get the `id` field from the request path parameter named `id`
              userId: $request.path.id
  # the path item of the linked operation
  /users/{userid}/address:
    parameters:
    - name: userid
      in: path
      required: true
      description: the user identifier, as userId
      schema:
        type: string
      # linked operation
      get:
        operationId: getUserAddress
        responses:
          '200':
            description: the user's address
```

When a runtime expression fails to evaluate, no parameter value is passed to the target operation.

Values from the response body can be used to drive a linked operation.

```yaml
links:
  address:
    operationId: getUserAddressByUUID
    parameters:
      # get the `id` field from the request path parameter named `id`
      userUuid: $response.body#/uuid
```

Clients follow all links at their discretion.
Neither permissions, nor the capability to make a successful call to that link, is guaranteed
solely by the existence of a relationship.

##### OperationRef Examples

As references to `operationId` MAY NOT be possible (the `operationId` is an optional
value), references MAY also be made through a relative `operationRef`:

```yaml
links:
  UserRepositories:
    # returns array of '#/components/schemas/repository'
    operationRef: '#/paths/~12.0~1repositories~1{username}/get'
    parameters:
      username: $response.body#/username
```

or an absolute `operationRef`:

```yaml
links:
  UserRepositories:
    # returns array of '#/components/schemas/repository'
    operationRef: 'https://na2.gigantic-server.com/#/paths/~12.0~1repositories~1{username}/get'
    parameters:
      username: $response.body#/username
```

Note that in the use of `operationRef`, the _escaped forward-slash_ is necessary when
using JSON references.

##### <a name="runtimeExpression"></a>Runtime Expressions

Runtime expressions allow defining values based on information that will only be available within the HTTP message in an actual API call.
This mechanism is used by [Link Objects](#linkObject) and [Callback Objects](#callbackObject).

The runtime expression is defined by the following [ABNF](https://tools.ietf.org/html/rfc5234) syntax

```text
      expression = ( "$url" | "$method" | "$statusCode" | "$request." source | "$response." source )
      source = ( header-reference | query-reference | path-reference | body-reference )
      header-reference = "header." token
      query-reference = "query." name
      path-reference = "path." name
      body-reference = "body" ["#" fragment]
      fragment = a JSON Pointer [RFC 6901](https://tools.ietf.org/html/rfc6901)
      name = *( char )
      char = as per RFC [7159](https://tools.ietf.org/html/rfc7159#section-7)
      token = as per RFC [7230](https://tools.ietf.org/html/rfc7230#section-3.2.6)
```

The `name` identifier is case-sensitive, whereas `token` is not.

The table below provides examples of runtime expressions and examples of their use in a value:

##### <a name="runtimeExpressionExamples"></a>Examples

Source Location | example expression  | notes
---|:---|:---|
HTTP Method            | `$method`         | The allowable values for the `$method` will be those for the HTTP operation.
Requested media type | `$request.header.accept`        |
Request parameter      | `$request.path.id`        | Request parameters MUST be declared in the `parameters` section of the parent operation or they cannot be evaluated. This includes request headers.
Request body property   | `$request.body#/user/uuid`   | In operations which accept payloads, references may be made to portions of the `requestBody` or the entire body.
Request URL            | `$url`            |
Response value         | `$response.body#/status`       |  In operations which return payloads, references may be made to portions of the response body or the entire body.
Response header        | `$response.header.Server` |  Single header values only are available

Runtime expressions preserve the type of the referenced value.
Expressions can be embedded into string values by surrounding the expression with `{}` curly braces.

#### <a name="headerObject"></a>Header 对象

Header 对象除了以下改动之外与 [Parameter 对象](#parameterObject) 一致：

1. `name` 不能被指定，它在相应的 `headers` 映射中被指定。
2. `in` 不能被指定，它隐含在 `header` 中。
3. 所有被 location 影响的特性必须适合 `header` 中的一个 location，(比如 [`style`](#parameterStyle))。

##### Header 对象示例

一个类型为 `integer` 的简单 header：

```json
{
  "description": "The number of allowed requests in the current period",
  "schema": {
    "type": "integer"
  }
}
```

```yaml
description: The number of allowed requests in the current period
schema:
  type: integer
```

#### <a name="tagObject"></a>Tag Object

Adds metadata to a single tag that is used by the [Operation Object](#operationObject).
It is not mandatory to have a Tag Object per tag defined in the Operation Object instances.

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="tagName"></a>name | `string` | **必选**. The name of the tag.
<a name="tagDescription"></a>description | `string` | A short description for the tag. [CommonMark syntax](http://spec.commonmark.org/)可以被用来呈现富文本格式.
<a name="tagExternalDocs"></a>externalDocs | [External Documentation Object](#externalDocumentationObject) | Additional external documentation for this tag.

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### Tag 对象示例

```json
{
  "name": "pet",
  "description": "Pets operations"
}
```

```yaml
name: pet
description: Pets operations
```

#### <a name="examplesObject"></a>Examples Object

In an `example`, a JSON Reference MAY be used, with the
explicit restriction that examples having a JSON format with object named
`$ref` are not allowed. Therefore, that `example`, structurally, can be
either a string primitive or an object, similar to `additionalProperties`.

In all cases, the payload is expected to be compatible with the type schema
for the associated value.  Tooling implementations MAY choose to
validate compatibility automatically, and reject the example value(s) if they
are incompatible.

```yaml
# in a model
schemas:
  properties:
    name:
      type: string
      example:
        $ref: http://foo.bar#/examples/name-example

# in a request body, note the plural `examples`
  requestBody:
    content:
      'application/json':
        schema:
          $ref: '#/components/schemas/Address'
        examples:
          foo:
            value: {"foo": "bar"}
          bar:
            value: {"bar": "baz"}
      'application/xml':
        examples:
          xml:
            externalValue: 'http://foo.bar/examples/address-example.xml'
      'text/plain':
        examples:
          text:
            externalValue: 'http://foo.bar/examples/address-example.txt'

# in a parameter
  parameters:
    - name: 'zipCode'
      in: 'query'
      schema:
        type: 'string'
        format: 'zip-code'
        example:
          $ref: 'http://foo.bar#/examples/zip-example'

# in a response, note the singular `example`:
  responses:
    '200':
      description: your car appointment has been booked
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/SuccessResponse'
          example:
            $ref: http://foo.bar#/examples/address-example.json
```

#### <a name="referenceObject"></a>Reference 对象

一个允许引用规范内部的其他部分或外部规范的对象。

Reference Object 定义于 [JSON Reference](https://tools.ietf.org/html/draft-pbryan-zyp-json-ref-03) 且遵循相同的结构、行为和规则。

For this specification, reference resolution is accomplished as defined by the JSON Reference specification and not by the JSON Schema specification.

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="referenceRef"></a>$ref | `string` | **必选**. 引用字符串。

此对象不能被扩展，任何附加的属性将会被忽略。

##### Reference 对象示例

```json
{
  "$ref": "#/components/schemas/Pet"
}
```

```yaml
$ref: '#/components/schemas/Pet'
```

##### 关联外部文档示例

```json
{
  "$ref": "Pet.json"
}
```

```yaml
$ref: Pet.yaml
```

##### 关联外部文档的一部分

```json
{
  "$ref": "definitions.json#/Pet"
}
```

```yaml
$ref: definitions.yaml#/Pet
```

#### <a name="schemaObject"></a>Schema Object

Schema Object 用于定义输入和输出的数据类型。这些类型可以是对象，但也可以是原始值和数组。这个对象是 [JSON Schema Specification Wright Draft 00](http://json-schema.org/) 扩展后的子集.

关于property的的更多信息请查看 [JSON Schema Core](https://tools.ietf.org/html/draft-wright-json-schema-00) 和 [JSON Schema Validation](https://tools.ietf.org/html/draft-wright-json-schema-validation-00)。除非另有说明，否则 properties 定义遵循JSON Schema。

##### Properties

以下 properties 是直接从 JSON Schema 提取出来的，而且遵循同样的规范：

- title
- multipleOf
- maximum
- exclusiveMaximum
- minimum
- exclusiveMinimum
- maxLength
- minLength
- pattern (This string SHOULD be a valid regular expression, according to the [ECMA 262 regular expression](https://www.ecma-international.org/ecma-262/5.1/#sec-7.8.5) dialect)
- maxItems
- minItems
- uniqueItems
- maxProperties
- minProperties
- required
- enum

以下 properties 是从 JSON Schema 提取出来的，但是做了一些调整以适应 OpenAPI Specification。

- type - 值必须是一个字符串，不支持以数组形式定义多个值。
- allOf - Inline 或 referenced 的 schema 必须是一个 [Schema Object](#schemaObject) 且不是一个标准的 JSON Schema。
- oneOf - Inline 或 referenced 的 schema 必须是一个 [Schema Object](#schemaObject) 且不是一个标准的 JSON Schema。
- anyOf - Inline 或 referenced 的 schema 必须是一个 [Schema Object](#schemaObject) 且不是一个标准的 JSON Schema。
- not - Inline 或 referenced 的 schema 必须是一个 [Schema Object](#schemaObject) 且不是一个标准的 JSON Schema。
- items - 值必须是一个对象且不是一个数组。Inline 或 referenced 的 schema 必须是一个 [Schema Object](#schemaObject)且不是一个标准的 JSON Schem。. `items` 必须存在如果 `type` 的值是 `array`。
- properties - Property 定义必须是一个 [Schema Object](#schemaObject) 且不是一个标准的 JSON Schema (inline 或 referenced).
- additionalProperties - 值可以是 boolean 或 object. Inline 或 referenced schema 必须是一个 [Schema Object](#schemaObject) 且不是一个标准的 JSON Schema。
- description - [CommonMark syntax](http://spec.commonmark.org/)可以被用来呈现富文本格式.
- format - 查看 [Data Type Formats](#dataTypeFormat) 以深入了解细节。在依靠 JSON Schema 定义的格式的同时，OAS 额外提供了一些预定义的格式。
- default - The default value represents what would be assumed by the consumer of the input as the value of the schema if one is not provided. 不同于 JSON Schema，这个值必须符合定义与相同级别的 Schema Object 中定义的类型，比如 `type` 是 `string`，那么 `default` 可以是 `"foo"` 但不能是 `1`。

另外，任何可以使用 Schema Object 的地方也可以使用 [Reference Object](#referenceObject) 替代。这允许引用一个定义而避免重复定义。

未在此处提及的 JSON Schema 规范中定义的其他属性将严格的不被支持。

Other than the JSON Schema subset fields, 以下字段可能会被用于后续的 schema documentation：

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="schemaNullable"></a>nullable | `boolean` | 对于定义的schema，允许发送 `null` 值。默认值是 `false`。
<a name="schemaDiscriminator"></a>discriminator | [Discriminator Object](#discriminatorObject) | Adds support for polymorphism. The discriminator is an object name that is used to differentiate between other schemas which may satisfy the payload description. See [Composition and Inheritance](#schemaComposition) for more details.
<a name="schemaReadOnly"></a>readOnly | `boolean` | 仅与 Schema `"properties"` 定义有关。 声明此属性是 "readonly" 的。这意味着它可以作为 response 的一部分但不应该作为 request 的一部分被发送。如果一个 property 的 `readOnly` 被标记为 `true` 且在 `required` 列表中，`required` 将只作用于 response。一个 property 的 `readOnly` 和 `writeOnly` 不允许同时被标记为 `true`。默认值是 `false`。
<a name="schemaWriteOnly"></a>writeOnly | `boolean` | 仅与 Schema `"properties"` 定义有关。声明此 property 为 "write only"。所以它可以作为 request 的一部分而不应该作为 response 的一部分被发送。如果一个 property 的 `writeOnly` 被标记为 `true` 且在 `required` 列表中，`required` 将只作用于 request。一个 property 的 `readOnly` 和 `writeOnly` 不能同时被标记为 `true`。默认值是 `false`。
<a name="schemaXml"></a>xml | [XML Object](#xmlObject) | 这只能用于 properties schemas，在 root schemas 中没有效果。Adds additional metadata to describe the XML representation of this property.
<a name="schemaExternalDocs"></a>externalDocs | [External Documentation Object](#externalDocumentationObject) | 此 schema 附加的外部文档。
<a name="schemaExample"></a>example | Any | 一个用于示范此 schema实例的示例，可以是任意格式。为了表达无法用 JSON 或 YAML 格式呈现的示例，可以使用 string 类型的值，且在必要的地方需要使用字符转义。
<a name="schemaDeprecated"></a> deprecated | `boolean` | 表示一个 schema 是废弃的，应该逐渐被放弃使用。默认值是 `false`.

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

###### <a name="schemaComposition"></a>Composition and Inheritance (Polymorphism)

The OpenAPI Specification allows combining and extending model definitions using the `allOf` property of JSON Schema, in effect offering model composition.
`allOf` takes an array of object definitions that are validated *independently* but together compose a single object.

While composition offers model extensibility, it does not imply a hierarchy between the models.
To support polymorphism, the OpenAPI Specification adds the `discriminator` field.
When used, the `discriminator` will be the name of the property that decides which schema definition validates the structure of the model.
As such, the `discriminator` field MUST be a required field.
There are are two ways to define the value of a discriminator for an inheriting instance.

- Use the schema name.
- Override the schema name by overriding the property with a new value. If a new value exists, this takes precedence over the schema name.
As such, inline schema definitions, which do not have a given id, *cannot* be used in polymorphism.

###### XML Modeling

The [xml](#schemaXml) property allows extra definitions when translating the JSON definition to XML.
The [XML Object](#xmlObject) contains additional information about the available options.

##### Schema 对象示例s

###### Primitive Sample

```json
{
  "type": "string",
  "format": "email"
}
```

```yaml
type: string
format: email
```

###### Simple Model

```json
{
  "type": "object",
  "required": [
    "name"
  ],
  "properties": {
    "name": {
      "type": "string"
    },
    "address": {
      "$ref": "#/components/schemas/Address"
    },
    "age": {
      "type": "integer",
      "format": "int32",
      "minimum": 0
    }
  }
}
```

```yaml
type: object
required:
- name
properties:
  name:
    type: string
  address:
    $ref: '#/components/schemas/Address'
  age:
    type: integer
    format: int32
    minimum: 0
```

###### Model with Map/Dictionary Properties

For a simple string to string mapping:

```json
{
  "type": "object",
  "additionalProperties": {
    "type": "string"
  }
}
```

```yaml
type: object
additionalProperties:
  type: string
```

For a string to model mapping:

```json
{
  "type": "object",
  "additionalProperties": {
    "$ref": "#/components/schemas/ComplexModel"
  }
}
```

```yaml
type: object
additionalProperties:
  $ref: '#/components/schemas/ComplexModel'
```

###### Model with Example

```json
{
  "type": "object",
  "properties": {
    "id": {
      "type": "integer",
      "format": "int64"
    },
    "name": {
      "type": "string"
    }
  },
  "required": [
    "name"
  ],
  "example": {
    "name": "Puma",
    "id": 1
  }
}
```

```yaml
type: object
properties:
  id:
    type: integer
    format: int64
  name:
    type: string
required:
- name
example:
  name: Puma
  id: 1
```

###### Models with Composition

```json
{
  "components": {
    "schemas": {
      "ErrorModel": {
        "type": "object",
        "required": [
          "message",
          "code"
        ],
        "properties": {
          "message": {
            "type": "string"
          },
          "code": {
            "type": "integer",
            "minimum": 100,
            "maximum": 600
          }
        }
      },
      "ExtendedErrorModel": {
        "allOf": [
          {
            "$ref": "#/components/schemas/ErrorModel"
          },
          {
            "type": "object",
            "required": [
              "rootCause"
            ],
            "properties": {
              "rootCause": {
                "type": "string"
              }
            }
          }
        ]
      }
    }
  }
}
```

```yaml
components:
  schemas:
    ErrorModel:
      type: object
      required:
      - message
      - code
      properties:
        message:
          type: string
        code:
          type: integer
          minimum: 100
          maximum: 600
    ExtendedErrorModel:
      allOf:
      - $ref: '#/components/schemas/ErrorModel'
      - type: object
        required:
        - rootCause
        properties:
          rootCause:
            type: string
```

###### Models with Polymorphism Support

```json
{
  "components": {
    "schemas": {
      "Pet": {
        "type": "object",
        "discriminator": {
          "propertyName": "petType"
        },
        "properties": {
          "name": {
            "type": "string"
          },
          "petType": {
            "type": "string"
          }
        },
        "required": [
          "name",
          "petType"
        ]
      },
      "Cat": {
        "description": "A representation of a cat. Note that `Cat` will be used as the discriminator value.",
        "allOf": [
          {
            "$ref": "#/components/schemas/Pet"
          },
          {
            "type": "object",
            "properties": {
              "huntingSkill": {
                "type": "string",
                "description": "The measured skill for hunting",
                "default": "lazy",
                "enum": [
                  "clueless",
                  "lazy",
                  "adventurous",
                  "aggressive"
                ]
              }
            },
            "required": [
              "huntingSkill"
            ]
          }
        ]
      },
      "Dog": {
        "description": "A representation of a dog. Note that `Dog` will be used as the discriminator value.",
        "allOf": [
          {
            "$ref": "#/components/schemas/Pet"
          },
          {
            "type": "object",
            "properties": {
              "packSize": {
                "type": "integer",
                "format": "int32",
                "description": "the size of the pack the dog is from",
                "default": 0,
                "minimum": 0
              }
            },
            "required": [
              "packSize"
            ]
          }
        ]
      }
    }
  }
}
```

```yaml
components:
  schemas:
    Pet:
      type: object
      discriminator:
        propertyName: petType
      properties:
        name:
          type: string
        petType:
          type: string
      required:
      - name
      - petType
    Cat:  ## "Cat" will be used as the discriminator value
      description: A representation of a cat
      allOf:
      - $ref: '#/components/schemas/Pet'
      - type: object
        properties:
          huntingSkill:
            type: string
            description: The measured skill for hunting
            enum:
            - clueless
            - lazy
            - adventurous
            - aggressive
        required:
        - huntingSkill
    Dog:  ## "Dog" will be used as the discriminator value
      description: A representation of a dog
      allOf:
      - $ref: '#/components/schemas/Pet'
      - type: object
        properties:
          packSize:
            type: integer
            format: int32
            description: the size of the pack the dog is from
            default: 0
            minimum: 0
        required:
        - packSize
```

#### <a name="discriminatorObject"></a>Discriminator 对象

当一个 request bodies 或 response payloads 可以是多种 schemas 时，可以使用一个 `discriminator` 对象来帮助序列化、反序列化和校验。  The discriminator is a specific object in a schema which is used to inform the consumer of the specification of an alternative schema based on the value associated with it.

当使用 discriminator 时，_inline_ schemas 不会被考虑。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="propertyName"></a>propertyName | `string` | **必选**. 在 payload 中表示 discriminator 值的属性的名称。
<a name="discriminatorMapping"></a> mapping | Map[`string`, `string`] | 一个映射 payload 中的值和 schema 名称或引用的对象。

discriminator 属性仅在与 `oneOf`, `anyOf`, `allOf` 这几个复合关键字之一一起使用时才合法.

在 OAS 3.0 中，一个 response payload 仅可以使用一种类型来描述：

```yaml
MyResponseType:
  oneOf:
  - $ref: '#/components/schemas/Cat'
  - $ref: '#/components/schemas/Dog'
  - $ref: '#/components/schemas/Lizard'
```

也就是说 payload _必须_ 且只能满足 `Cat`、`Dog` 或 `Lizzard` schemas 中的一个。 In this case, a discriminator MAY act as a "hint" to shortcut validation and selection of the matching schema which may be a costly operation, depending on the complexity of the schema. We can then describe exactly which field tells us which schema to use:

```yaml
MyResponseType:
  oneOf:
  - $ref: '#/components/schemas/Cat'
  - $ref: '#/components/schemas/Dog'
  - $ref: '#/components/schemas/Lizard'
  discriminator:
    propertyName: pet_type
```

The expectation now is that a property with name `pet_type` _MUST_ be present in the response payload, and the value will correspond to the name of a schema defined in the OAS document.  Thus the response payload:

```json
{
  "id": 12345,
  "pet_type": "Cat"
}
```

Will indicate that the `Cat` schema be used in conjunction with this payload.

In scenarios where the value of the discriminator field does not match the schema name or implicit mapping is not possible, an optional `mapping` definition MAY be used:

```yaml
MyResponseType:
  oneOf:
  - $ref: '#/components/schemas/Cat'
  - $ref: '#/components/schemas/Dog'
  - $ref: '#/components/schemas/Lizard'
  - $ref: 'https://gigantic-server.com/schemas/Monster/schema.json'
  discriminator:
    propertyName: pet_type
    mapping:
      dog: '#/components/schemas/Dog'
      monster: 'https://gigantic-server.com/schemas/Monster/schema.json'
```

Here the discriminator _value_ of `dog` will map to the schema `#/components/schemas/Dog`, rather than the default (implicit) value of `Dog`.  If the discriminator _value_ does not match an implicit or explicit mapping, no schema can be determined and validation SHOULD fail. Mapping keys MUST be string values, but tooling MAY convert response values to strings for comparison.

When used in conjunction with the `anyOf` construct, the use of the discriminator can avoid ambiguity where multiple schemas may satisfy a single payload.

In both the `oneOf` and `anyOf` use cases, all possible schemas MUST be listed explicitly.  To avoid redundancy, the discriminator MAY be added to a parent schema definition, and all schemas comprising the parent schema in an `allOf` construct may be used as an alternate schema.

For example:

```yaml
components:
  schemas:
    Pet:
      type: object
      required:
      - pet_type
      properties:
        pet_type:
          type: string
      discriminator:
        propertyName: pet_type
        mapping:
          cachorro: Dog
    Cat:
      allOf:
      - $ref: '#/components/schemas/Pet'
      - type: object
        # all other properties specific to a `Cat`
        properties:
          name:
            type: string
    Dog:
      allOf:
      - $ref: '#/components/schemas/Pet'
      - type: object
        # all other properties specific to a `Dog`
        properties:
          bark:
            type: string
    Lizard:
      allOf:
      - $ref: '#/components/schemas/Pet'
      - type: object
        # all other properties specific to a `Lizard`
        properties:
          lovesRocks:
            type: boolean
```

a payload like this:

```json
{
  "pet_type": "Cat",
  "name": "misty"
}
```

will indicate that the `Cat` schema be used.  Likewise this schema:

```json
{
  "pet_type": "cachorro",
  "bark": "soft"
}
```

will map to `Dog` because of the definition in the `mappings` element.

#### <a name="xmlObject"></a>XML 对象

一个为 XML 模型定义微调过的元数据对象。

当使用数组时，不会推测 XML 元素的名称（单数或复数形式），所以应该添加 `name` 属性来提供此信息。
查看展示此期望的示例。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="xmlName"></a>name | `string` | 替换用于描述元素/属性的结构特性的名称。当在 `items` 内定义时将会影响处于此列表中的每个元素的名称。当定义于 `items` 之上时将会影响它说包裹的元素且仅当 `wrapped` 是 `true` 时，如果 `wrapped` 是 `false` 时它将会被忽略。
<a name="xmlNamespace"></a>namespace | `string` | 命名空间定义的 URI。其值必须是绝对 URI。
<a name="xmlPrefix"></a>prefix | `string` | 用于 [name](#xmlName) 的前缀。
<a name="xmlAttribute"></a>attribute | `boolean` | 声明此特性定义会被转换为一个属性而不是一个元素。默认值是 `false`。
<a name="xmlWrapped"></a>wrapped | `boolean` | 只可被用于数组定义。表示数组是否被包裹（比如, `<books><book/><book/></books>`）或未被包裹（`<book/><book/>`）。默认值是 `false`。此定义只在 `type` 为 `array`（位于 `items` 之上） 时生效。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### XML 对象示例

XML 对象定义的示例被包括在一个 [Schema 对象](#schemaObject) 的特性定义并带有一个样例 XML 来呈现它。

###### 无 XML 元素

基础字符串属性：

```json
{
    "animals": {
        "type": "string"
    }
}
```

```yaml
animals:
  type: string
```

```xml
<animals>...</animals>
```

基础字符串数组属性 ([`wrapped`](#xmlWrapped) 默认是 `false`)：

```json
{
    "animals": {
        "type": "array",
        "items": {
            "type": "string"
        }
    }
}
```

```yaml
animals:
  type: array
  items:
    type: string
```

```xml
<animals>...</animals>
<animals>...</animals>
<animals>...</animals>
```

###### XML 名称替换

```json
{
  "animals": {
    "type": "string",
    "xml": {
      "name": "animal"
    }
  }
}
```

```yaml
animals:
  type: string
  xml:
    name: animal
```

```xml
<animal>...</animal>
```

###### XML 属性，前缀和命名空间

在此示例中展示了一个完整的模型定义。

```json
{
  "Person": {
    "type": "object",
    "properties": {
      "id": {
        "type": "integer",
        "format": "int32",
        "xml": {
          "attribute": true
        }
      },
      "name": {
        "type": "string",
        "xml": {
          "namespace": "http://example.com/schema/sample",
          "prefix": "sample"
        }
      }
    }
  }
}
```

```yaml
Person:
  type: object
  properties:
    id:
      type: integer
      format: int32
      xml:
        attribute: true
    name:
      type: string
      xml:
        namespace: http://example.com/schema/sample
        prefix: sample
```

```xml
<Person id="123">
    <sample:name xmlns:sample="http://example.com/schema/sample">example</sample:name>
</Person>
```

###### XML 数组

改变元素的名称：

```json
{
  "animals": {
    "type": "array",
    "items": {
      "type": "string",
      "xml": {
        "name": "animal"
      }
    }
  }
}
```

```yaml
animals:
  type: array
  items:
    type: string
    xml:
      name: animal
```

```xml
<animal>value</animal>
<animal>value</animal>
```

外部的 `name` 属性在 XML 上不产生效果：

```json
{
  "animals": {
    "type": "array",
    "items": {
      "type": "string",
      "xml": {
        "name": "animal"
      }
    },
    "xml": {
      "name": "aliens"
    }
  }
}
```

```yaml
animals:
  type: array
  items:
    type: string
    xml:
      name: animal
  xml:
    name: aliens
```

```xml
<animal>value</animal>
<animal>value</animal>
```

尽管数组是被包裹的，如果没有明确定义一个名称，那么同样地名称会被用于内部和外部：

```json
{
  "animals": {
    "type": "array",
    "items": {
      "type": "string"
    },
    "xml": {
      "wrapped": true
    }
  }
}
```

```yaml
animals:
  type: array
  items:
    type: string
  xml:
    wrapped: true
```

```xml
<animals>
  <animals>value</animals>
  <animals>value</animals>
</animals>
```

为了解决上面示例的命名问题，可以使用下面的定义：

```json
{
  "animals": {
    "type": "array",
    "items": {
      "type": "string",
      "xml": {
        "name": "animal"
      }
    },
    "xml": {
      "wrapped": true
    }
  }
}
```

```yaml
animals:
  type: array
  items:
    type: string
    xml:
      name: animal
  xml:
    wrapped: true
```

```xml
<animals>
  <animal>value</animal>
  <animal>value</animal>
</animals>
```

Affecting both internal and external names:

```json
{
  "animals": {
    "type": "array",
    "items": {
      "type": "string",
      "xml": {
        "name": "animal"
      }
    },
    "xml": {
      "name": "aliens",
      "wrapped": true
    }
  }
}
```

```yaml
animals:
  type: array
  items:
    type: string
    xml:
      name: animal
  xml:
    name: aliens
    wrapped: true
```

```xml
<aliens>
  <animal>value</animal>
  <animal>value</animal>
</aliens>
```

如果我们改变外部的元素而保持内部的不变：

```json
{
  "animals": {
    "type": "array",
    "items": {
      "type": "string"
    },
    "xml": {
      "name": "aliens",
      "wrapped": true
    }
  }
}
```

```yaml
animals:
  type: array
  items:
    type: string
  xml:
    name: aliens
    wrapped: true
```

```xml
<aliens>
  <aliens>value</aliens>
  <aliens>value</aliens>
</aliens>
```

#### <a name="securitySchemeObject"></a>Security Scheme 对象

定义一个用于 operations 的 security scheme。被支持的 schemes 有 HTTP 认证，一个 API key（作为 header 或 query parameter），定义于[RFC6749](https://tools.ietf.org/html/rfc6749) 的 Oauth2 常用流程（implicit、password、application 和 access code）和 [OpenID Connect Discovery](https://tools.ietf.org/html/draft-ietf-oauth-discovery-06)。

##### 固定字段

字段名 | 类型 | Applies To | 描述
---|:---:|---|---
<a name="securitySchemeType"></a>type | `string` | Any | **必选**. security scheme 的类型。有效值包括 `"apiKey"`, `"http"`, `"oauth2"`, `"openIdConnect"`.
<a name="securitySchemeDescription"></a>description | `string` | Any | 对 security scheme 的简短描述. [CommonMark syntax](http://spec.commonmark.org/)可以被用来呈现富文本格式.
<a name="securitySchemeName"></a>name | `string` | `apiKey` | **必选**. 用于 header、 query 或 cookie 的参数名字。
<a name="securitySchemeIn"></a>in | `string` | `apiKey` | **必选**. API key 的位置。有效值包括 `"query"`、`"header"` 或 `"cookie"`.
<a name="securitySchemeScheme"></a>scheme | `string` | `http` | **必选**. 用于 [Authorization header as defined in RFC7235](https://tools.ietf.org/html/rfc7235#section-5.1) 的 HTTP Auahorization scheme 的名字.
<a name="securitySchemeBearerFormat"></a>bearerFormat | `string` | `http` (`"bearer"`) | 用于提示客户端所使用的bearer token的格式。Bearer token 通常通过一个authorization server生成，所以这个字段最主要的目的是用来记录这个信息。
<a name="securitySchemeFlows"></a>flows | [OAuth Flows Object](#oauthFlowsObject) | `oauth2` | **必选**. 一个包含所支持的 flow types 的配置信息的对象。
<a name="securitySchemeOpenIdConnectUrl"></a>openIdConnectUrl | `string` | `openIdConnect` | **必选**. 用于发现 OAuth2 配置值的OpenId Connect URL，必须是 URL 形式。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### Security Scheme 对象示例

###### Basic Authentication Sample

```json
{
  "type": "http",
  "scheme": "basic"
}
```

```yaml
type: http
scheme: basic
```

###### API Key Sample

```json
{
  "type": "apiKey",
  "name": "api_key",
  "in": "header"
}
```

```yaml
type: apiKey
name: api_key
in: header
```

###### JWT Bearer Sample

```json
{
  "type": "http",
  "scheme": "bearer",
  "bearerFormat": "JWT",
}
```

```yaml
type: http
scheme: bearer
bearerFormat: JWT
```

###### Implicit OAuth2 Sample

```json
{
  "type": "oauth2",
  "flows": {
    "implicit": {
      "authorizationUrl": "https://example.com/api/oauth/dialog",
      "scopes": {
        "write:pets": "modify pets in your account",
        "read:pets": "read your pets"
      }
    }
  }
}
```

```yaml
type: oauth2
flows:
  implicit:
    authorizationUrl: https://example.com/api/oauth/dialog
    scopes:
      write:pets: modify pets in your account
      read:pets: read your pets
```

#### <a name="oauthFlowsObject"></a>OAuth Flows 对象

允许配置支持的 OAuth Flows。

##### 固定字段

字段名 | 类型 | 描述
---|:---:|---
<a name="oauthFlowsImplicit"></a>implicit| [OAuth Flow Object](#oauthFlowObject) | OAuth Implicit flow 的配置
<a name="oauthFlowsPassword"></a>password| [OAuth Flow Object](#oauthFlowObject) | OAuth Resource Owner Password flow 的配置
<a name="oauthFlowsClientCredentials"></a>clientCredentials| [OAuth Flow Object](#oauthFlowObject) | OAuth Client Credentials flow 的配置。在 OpenAPI 2.0 中曾被称作 `application`。
<a name="oauthFlowsAuthorizationCode"></a>authorizationCode| [OAuth Flow Object](#oauthFlowObject) | OAuth Authorization Code flow 的配置。在 OpenAPI 2.0 中曾被称作 `accessCode`。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

#### <a name="oauthFlowObject"></a>OAuth Flow 对象

支持的 OAuth Flow 的配置详情。

##### 固定字段

字段名 | 类型 | Applies To | 描述
---|:---:|---|---
<a name="oauthFlowAuthorizationUrl"></a>authorizationUrl | `string` | `oauth2` (`"implicit"`, `"authorizationCode"`) | **必选**。用于此流程的 authorization URL。必须是 URL 形式。
<a name="oauthFlowTokenUrl"></a>tokenUrl | `string` | `oauth2` (`"password"`, `"clientCredentials"`, `"authorizationCode"`) | **必选**。用于此流程的 token URL。必须是 URL 形式。
<a name="oauthFlowRefreshUrl"></a>refreshUrl | `string` | `oauth2` | 用于获取 refresh tokens 的 URL，必须是 URL 形式。
<a name="oauthFlowScopes"></a>scopes | Map[`string`, `string`] | `oauth2` | **必选**。可用于 OAuth2 security scheme 的 scope。scope 名称与其简短描述的映射。

这个对象可能会被[规范扩展](#specificationExtensions)扩展。

##### OAuth Flow 对象示例s

```JSON
{
  "type": "oauth2",
  "flows": {
    "implicit": {
      "authorizationUrl": "https://example.com/api/oauth/dialog",
      "scopes": {
        "write:pets": "modify pets in your account",
        "read:pets": "read your pets"
      }
    },
    "authorizationCode": {
      "authorizationUrl": "https://example.com/api/oauth/dialog",
      "tokenUrl": "https://example.com/api/oauth/token",
      "scopes": {
        "write:pets": "modify pets in your account",
        "read:pets": "read your pets"
      }
    }
  }
}
```

```YAML
type: oauth2
flows:
  implicit:
    authorizationUrl: https://example.com/api/oauth/dialog
    scopes:
      write:pets: modify pets in your account
      read:pets: read your pets
  authorizationCode:
    authorizationUrl: https://example.com/api/oauth/dialog
    tokenUrl: https://example.com/api/oauth/token
    scopes:
      write:pets: modify pets in your account
      read:pets: read your pets
```

#### <a name="securityRequirementObject"></a>Security Requirement 对象

列出执行此 operation 所需的 security schemes。每个属性的名字都必须与
 [Components Object](#componentsObject) 中 [Security Schemes](#componentsSecuritySchemes) 声明的 security scheme 相符。

包含多个 schemes 的 Security Requirement 对象中的所有 scheme 都必须要满足授权请求。这便能够支持需要使用多个 query parameters 或 HTTP headers 来传递安全信息的情景。

当When a list of Security Requirement Objects is defined on the [Open API 对象](#oasObject) 或 [Operation 对象] (#operationObject) 包含一组 Security Requirement 对象时，请求只需要满足其中一个即可。

##### 模式字段

字段名模式 | 类型 | 描述
---|:---:|---
<a name="securityRequirementsName"></a>{name} | [`string`] | 每个名称都必须对应于 [Components 对象](#componentsObject) 下的 [Security Schemes](#componentsSecuritySchemes) 的一个 security scheme。如果此 security scheme 是 `"oauth2"` 或 `"openIdConnect"` 类型，那么其值是用于执行的一组 scope names。对于其他 security scheme 类型。此数组必须是空的。

##### Security Requirement 对象示例

###### Non-OAuth2 Security Requirement

```json
{
  "api_key": []
}
```

```yaml
api_key: []
```

###### OAuth2 Security Requirement

```json
{
  "petstore_auth": [
    "write:pets",
    "read:pets"
  ]
}
```

```yaml
petstore_auth:
- write:pets
- read:pets
```

### <a name="specificationExtensions"></a>规范扩展

尽管 OpenAPI Specification 尝试包含大部分的使用场景，在需要时仍然可以通过附加数据来扩展此规范。

此扩展属性被设计为总是以 `"x-"` 为前缀的模式字段。

字段名模式 | 类型 | 描述
---|:---:|---
<a name="infoExtensions"></a>^x- | Any | 允许扩展 OpenAPI Schema。此字段名必须以 `x-` 开头，比如 `x-internal-id`。此字段的值可以是 `null`、原始类型、数组或对象。可以包含任意有效的 JSON 格式的值。

此扩展可能会也可能不会被当前的工具支持，但是可以请求工具开发者支持此扩展（如果工具是内部或开源的）。

### <a name="securityFiltering"></a>Security Filtering

Some objects in the OpenAPI Specification MAY be declared and remain empty, or be completely removed, even though they are inherently the core of the API documentation.

The reasoning is to allow an additional layer of access control over the documentation.
While not part of the specification itself, certain libraries MAY choose to allow access to parts of the documentation based on some form of authentication/authorization.

Two examples of this:

1. The [Paths Object](#pathsObject) MAY be empty. It may be counterintuitive, but this may tell the viewer that they got to the right place, but can't access any documentation. They'd still have access to the [Info Object](#infoObject) which may contain additional information regarding authentication.
2. The [Path Item Object](#pathItemObject) MAY be empty. In this case, the viewer will be aware that the path exists, but will not be able to see any of its operations or parameters. This is different than hiding the path itself from the [Paths Object](#pathsObject), so the user will not be aware of its existence. This allows the documentation provider to finely control what the viewer can see.

## <a name="revisionHistory"></a>Appendix A: Revision History

Version   | Date       | Notes
---       | ---        | ---
3.0.0     | 2017-07-26 | Release of the OpenAPI Specification 3.0.0
3.0.0-rc2 | 2017-06-16 | rc2 of the 3.0 specification
3.0.0-rc1 | 2017-04-27 | rc1 of the 3.0 specification
3.0.0-rc0 | 2017-02-28 | Implementer's Draft of the 3.0 specification
2.0       | 2015-12-31 | Donation of Swagger 2.0 to the Open API Initiative
2.0       | 2014-09-08 | Release of Swagger 2.0
1.2       | 2014-03-14 | Initial release of the formal document.
1.1       | 2012-08-22 | Release of Swagger 1.1
1.0       | 2011-08-10 | First release of the Swagger Specification
