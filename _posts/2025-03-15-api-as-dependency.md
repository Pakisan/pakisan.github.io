---
title: Interpreting API as a full-fledged dependency
description: TODO
date: 2025-03-13 10:36:44 +0400
author: pakisan
categories: [Specifications, API]
tags: [asyncapi, openapi]
---

## Introduction

Like any developer, I have worked with APIs for a long time. I had a standard set of tools that was enough for me:
- OpenAPI specification for documenting asynchronous APIs and their asynchronous callbacks, long before they appeared in version 3.0
- Postman Collections to share API calls with colleagues inside and outside my team
- Swagger UI for visual representation of the API

Everything changed when I joined a private bank in 2019. Initially, I was responsible for developing a chat system for individual and corporate clients,
and later for the premium segment and investors. This introduced a new combination of WebSocket, SSE, and REST API. At that moment, my usual set of tools stopped working

That’s how I discovered AsyncAPI. I started implementing it in the chat system and gradually developed a library for generating AsyncAPI documentation through Java code, 
along with a JetBrains IDE plugin. Over four and a half years of working with AsyncAPI, I completely re-evaluated my approach to APIs and started perceiving them in a completely different way

Companies changed, but the same fundamental problems remained:
- Debates over code-first vs. API-first approaches
- Documentation being stored in various places, from Confluence Wiki to API portals

But rarely did people think of APIs as a product. That’s why, before moving on to the main idea, I want to emphasize this statement.

### Statements

#### API is a reason why we are writing services

A service, by itself, is not needed by anyone

It implements the concept of a provided service, which is defined through an API contract

The specific specification used — OpenAPI, AsyncAPI, RAML, etc. — does not matter

#### API is a product

If an API is the reason a service exists, then API itself is a full-fledged product with its own lifecycle

Users don’t care about our service’s architecture or technology stack

It doesn’t matter whether the user is internal or external—what matters to them is:
- How fast our API responds at p50, p90
- What its SLA is
- How many requests it can handle at once
- How much of their required functionality it can cover

For me, the last point is the most important. At its core, API is about functional abstraction — something that can be 
reused when developing a new service, reducing development costs

After all, [developing an API is expensive](https://apievangelist.com/2024/11/27/how-much-does-it-cost-to-develop-an-api/)

## API as a full-fledged dependency

An API is a dependency that I bring into my project to integrate with a particular system. And I want to work with it like any other dependency:
- Know where to find it:
  - NPM
  - Maven Central
  - PyPi
  - RubyGems
  - etc
- Receive notifications about new versions, vulnerabilities, and their end-of-life (EOL)
- Know where to look to understand what the service consists of

How can we achieve this?

### API discoverability

A single standard for accessing an API catalog—api-catalog—could work

To organize it effectively, it's important to be aware of one specification and one RFC

#### RFC - api-catalog: a well-known URI and link relation to help discovery of APIs

This [RFC: HTTP API Catalog](https://datatracker.ietf.org/doc/draft-ietf-httpapi-api-catalog/08/) was written by Kevin Smith

It proposes introducing a new [Well-Known](https://en.wikipedia.org/wiki/Well-known_URI) URL to retrieve the API catalog: `/.well-known/api-catalog`

```shell
curl -i --location 'https://www.example.com/.well-known/api-catalog'
HTTP/1.1 200
Content-Type: application/linkset+json

{
  "linkset": [
    {
      "anchor": "https://www.example.com/.well-known/api-catalog",
      "item": [
        {
          "href": "https://developer.example.com/apis/foo_api"
        },
        {
          "href": "https://developer.example.com/apis/bar_api"
        },
        {
          "href": "https://developer.example.com/apis/cantona_api"
        }
      ],
      "api-catalog": "https://www.example.net/.well-known/api-catalog"
    }
  ]
}
```

This approach will be familiar to those who have worked with [OpenID Connect(OIDC)](https://openid.net) since it uses a similar method for discovering configurations — `/.well-known/openid-configuration` — as described in [RFC: OpenID Connect Discovery 1.0](https://openid.net/specs/openid-connect-discovery-1_0.html)

#### APIs.json Specification

This [specification](https://apisjson.org) was written by [Kin Lane](https://www.linkedin.com/in/kinlane/). It proposes the following approach:

We have a URL — `/apis.json` — where we can retrieve all the necessary information about an API:
- Documentation
- Terms of use
- Contracts
- Owners
- etc.

```shell
curl -i --location 'https://www.example.com/apis.json'
HTTP/1.1 200
Content-Type: application/json

{
  "aid": "apis.json",
  "name": "Example API",
  "type": "Index",
  "description": "This is an example APIs.json file, demonstrating what is possible with the API discovery specification.",
  "image": "https://kinlane-productions.s3.amazonaws.com/apis-json/apis-json-logo.jpg",
  "tags": [
    "Application Programming Interface",
    "API"
  ],
  "created": "2024-11-06",
  "modified": "2024-11-06",
  "url": "https://example.com/apis.json",
  "specificationVersion": "0.19",
  "apis": [
    {
      "aid": "apis.json:example-api",
      "name": "Example API",
      "description": "This provides details about a specific API, telling what is possible.",
      "image": "https://kinlane-productions.s3.amazonaws.com/apis-json/apis-json-logo.jpg",
      "humanURL": "http://example.com",
      "baseURL": "http://api.example.com",
      "tags": [
        "API",
        "Application Programming Interface"
      ],
      "properties": [
        {
          "type": "Documentation",
          "url": "https://example.com/documentation"
        },
        {
          "type": "OpenAPI",
          "url": "https://example.com/openapi.yml"
        },
        {
          "type": "AsyncAPI",
          "url": "https://example.com/asyncapi.yml"
        }
      ],
      "overlays": [
        {
          "type": "APIs.io Search",
          "url": "https://example.com/apis-io-search"
        },
        {
          "type": "API Evangelist Rating",
          "url": "http://example.com/api-evangelist-ratings"
        }
      ],
      "contact": [
        {
          "FN": "APIs.json",
          "email": "info@apisjson.org"
        }
      ]
    }
  ],
  "maintainers": [
    {
      "FN": "Kin Lane",
      "X-twitter": "apievangelist",
      "email": "info@apievangelist.com"
    }
  ]
}
```

This approach can be compared to the [Sitemaps Protocol](https://www.sitemaps.org/protocol.html). We have enough information to index the entire APIs

---

Regardless of how our API is stored, we have a way to retrieve a catalog for further work:
- https://www.example.com/.well-known/api-catalog
- https://github.com/example-org/api/.well-known/api-catalog
- https://www.example.com/apis.json
- https://github.com/example-org/api/apis.json

Now that we understand how to organize API discoverability, we can move forward

### Getting it all together

> All commands, their parameters, file structures, and APIs are for informational purposes only and will change during development
{:.prompt-warning}

#### Storing Dependencies in the Project

> Yes, I know the name is similar to `package.json` and overlaps with `apis.json` from the specification above, but I haven't come up with a better name yet 😬
{:.prompt-info}

```shell
➜ git:(main) ✗ ls -p | grep -v /
api.yaml
```

At the root of the project, we will have an `api.yaml` file to store information about the APIs used and their sources. Its structure is based on the `<repositories></repositories>` block from [Maven pom.xml](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html)

##### Manually Adding Dependencies

First, let's create a file

```shell
touch api.yaml
```

Now, let's add a list of API catalogs where we will search for APIs

```yaml
api-catalogs:
  - id: central
    url: https://saas-api-hub.com/.well-known/api-catalog
  - id: myCompanyName_chat-ai
    url: https://github.com/myCompanyName/ai-apis/tree/main/.well-known/api-catalog
  - id: myCompanyName_chat
    url: https://github.com/myCompanyName/chat-apis/tree/main/.well-known/api-catalog
```
{:file='api.yaml'}

Once the catalogs are listed, we can specify the APIs we are using

```yaml
apis:
  - groupId: com.github.myCompanyName
    type: asyncapi
    apiId: chat-api-gateway
    version: 2.34.0
```
{:file='api.yaml'}

If we are only interested in specific operations, we have the option to specify only those

```yaml
apis:
  - groupId: com.github.myCompanyName
    type: openapi
    apiId: chat-api-ai-assistant
    version: 1.3.0
    operationIds:
      - aiGenerateChatBlock
      - aiGetChatBlockSchemas
```
{:file='api.yaml'}

Now that we have listed the required APIs, we can verify the correctness of the dependencies

```shell
➜ git:(main) ✗ cat api.yaml
api-catalogs:
  - id: central
    url: https://saas-api-hub.com/.well-known/api-catalog
  - id: myCompanyName_chat-ai
    url: https://github.com/myCompanyName/ai-apis/tree/main/.well-known/api-catalog
  - id: myCompanyName_chat
    url: https://github.com/myCompanyName/chat-apis/tree/main/.well-known/api-catalog

apis:
  - groupId: com.github.myCompanyName
    type: asyncapi
    apiId: chat-api-gateway
    version: 2.34.0
  - groupId: com.github.myCompanyName
    type: openapi
    apiId: chat-api-ai-assistant
    version: 1.3.0
    operationIds:
      - aiGenerateChatBlock
      - aiGetChatBlockSchemas
```
{:file='api.yaml'}

##### Adding Dependencies via CLI

First, let's initialize the project

```shell
➜ git:(main) ✗ apim init
```

Here, we will add the entire API

```shell
➜ git:(main) ✗ apim install com.github.myCompanyName:asyncapi:chat-api-gateway:2.34.0
```

And here, we will list only the necessary operations

```shell
➜ git:(main) ✗ apim install com.github.myCompanyName:openapi:chat-api-ai-assistant:1.3.0:aiGenerateChatBlock
➜ git:(main) ✗ apim install com.github.myCompanyName:openapi:chat-api-ai-assistant:1.3.0:aiGetChatBlockSchemas
```

Now, let's check our dependencies

```shell
➜ git:(main) ✗ apim ls
demo-project@1.6.0 /Users/pavelbodyachevskiy/IdeaProjects/demo-project
├── com.github.myCompanyName:asyncapi:chat-api-gateway:2.34.0
├─┬ com.github.myCompanyName:openapi:chat-api-ai-assistant:1.3.0
│ ├── aiGenerateChatBlock
│ ├── aiGetChatBlockSchemas
```

#### Working with Dependencies in the Project

У нас есть `api.yaml`, в котором перечислены интересующие API. А это значит, пришло время выжать из них по максимуму

Наша главная задача, переиспользовать указанные API, для сокращения времени, необходимого на разработку нового сервиса. В этом нам поможет генерация кода из API, для заявленных интеграций

#### Генерация проекта с нуля

Если проект новый, то у нас есть возможность сгенерировать проект с нуля

```shell
➜ git:(main) ✗ apim generate-jvm-project \
  --groupId com.example \
  --artifactId demo-project \
  --version 1.6.0 \
  --language java \
  --framework spring-boot \
  --build-tool maven \
  --http-client feign \
  --use-dependencies true
```

После выполнения команды, получим скелет проекта

```shell
find . -type f
./src/main/java/com.example/inegrations/com.github.myCompanyName/asyncapi/chat-api-gateway/v_2_34_0/СhatApiGatewayListener.java
./src/main/java/Application.java
./gitignore
./api.yaml
./pom.xml
./README.md
```

Из-за отсутствия опубликованной зависимости в artifactory для `com.github.myCompanyName:asyncapi:chat-api-gateway:2.34.0`, был сгенерирован клиент внутри приложения

В то время как для `com.github.myCompanyName:openapi:chat-api-ai-assistant` будет пере использована уже существующая зависимость

```shell
cat pom.xml
<dependencies>
    <!-- Chat API AI Assistant Client -->
    <dependency>
        <groupId>com.github.myCompanyName</groupId>
        <artifactId>chat-api-ai-assistant-client</artifactId>
        <version>1.3.0</version>
    </dependency>
</dependencies>
```

Теперь пришло время заглянуть в `api.yaml`

```shell
➜ git:(main) ✗ cat api.yaml
build-info:
  jvm:
    language: java
    framework: spring-boot
    build-tool: maven
    build-file: ./pom.xml
    http-client: feign
  dependencies-has-priority: true
```
{:file='api.yaml'}

Появился новый блок `build-info`, который будет использоваться для:
- Генерации кода, если API не предоставляет готового клиента
- Поиска сборочного файла, для добавления или обновления зависимостей

Теперь, когда есть информация о необходимой реализации, можно собирать проект

```shell
➜ git:(main) ✗ apim build
```

#### Работа с существующим проектом

