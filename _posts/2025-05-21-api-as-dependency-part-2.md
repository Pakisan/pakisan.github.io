---
title: Package URL for storing API artifacts
description: Unifying the storage of required packages for APIs
date: 2025-05-21 01:36:44 +0400
author: pakisan
categories: [Specifications, API]
tags: [asyncapi, openapi]
mermaid: true
image:
  path: /assets/assets/img/2025-03-15-api-as-dependency/cover image.png
---

While working on unifying the storage of required packages for APIs, I found an interesting standard called [PURL](https://github.com/package-url/purl-spec) (Package URL)

## What is PURL?

Package URL (PURL) is a standardized way to identify and locate software packages across different ecosystems. It provides a consistent format for referencing packages regardless of their origin or package manager. The PURL specification defines a string format that contains all the necessary components to precisely identify a package:

```
pkg:type/namespace/name@version?qualifiers#subpath
```

## Benefits of PURL

- **Uniformity**: Provides a consistent way to reference packages across different ecosystems (npm, Maven, PyPI, etc.)
- **Simplicity**: Easy to read, parse, and construct
- **Flexibility**: Works with various package managers and repositories
- **Interoperability**: Facilitates communication between different systems and tools

In the context of API management, PURL allows us to significantly reduce the API manifest size and improve readability:

From this

```yaml
urn: com.myCompanyName:asyncapi:chat-api-gateway:2.34.0
cycle: Release
contract:
  $ref: 'https://api-hub.com/apis/asyncapi/5a0052fd-fd49-4eb1-a257-1f246906a0da'
artifacts:
  maven:
    - language: java
      framework: any # spring-boot | quarkus | micronaut
      dependency: # com.github.myCompanyName:chat-api-gateway:2.34.0
        groupId: com.myCompanyName
        artifactId: chat-api-gateway
        version: 2.34.0
  npm:
    - language: typescript
      dependency: # "@myCompanyName/chat-api-gateway": "2.34.0"
        groupId: myCompanyName
        artifactId: chat-api-gateway
        version: 2.34.0
```
{:file='API'}

To this

```yaml
urn: com.myCompanyName:asyncapi:chat-api-gateway:2.34.0
cycle: Release
contract:
  $ref: 'https://api-hub.com/apis/asyncapi/5a0052fd-fd49-4eb1-a257-1f246906a0da'
artifacts:
  - pkg:maven/com.myCompanyName/chat-api-gateway@2.34.0
  - pkg:npm/@myCompanyName/chat-api-gateway@2.34.0
```
{:file='API'}


Quick reminder, that by treating APIs as full-fledged dependencies with standardized references like PURL, teams can manage their API dependencies with the same rigor and tooling as they do with their software libraries
