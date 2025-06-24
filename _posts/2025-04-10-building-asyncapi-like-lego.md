---
title: Building AsyncAPI contracts like lego | Modular API Design
description: Learn how to split AsyncAPI contracts into reusable components while maintaining auto-completion and validation capabilities. This guide introduces the x-component extension for modular API design
date: 2025-04-11 20:28:44 +0400
author: pakisan
categories: [Specifications, API]
tags: [asyncapi, openapi]
head:
  - - meta
    - name: keywords
      content: asyncapi components, modular api design, api reusability, openapi components, x-component extension, api contract validation, component extraction, api standardization
  - - meta
    - property: og:title
      content: Building AsyncAPI contracts like lego | Modular API Design
  - - meta
    - property: og:description
      content: Learn how to split AsyncAPI contracts into reusable components while maintaining auto-completion and validation capabilities. This guide introduces the x-component extension for modular API design
  - - meta
    - property: og:type
      content: article
  - - meta
    - property: og:url
      content: https://pavelon.dev/posts/building-asyncapi-like-lego/
  - - meta
    - name: twitter:title
      content: Building AsyncAPI contracts like lego | Modular API Design
  - - meta
    - name: twitter:description
      content: Learn how to split AsyncAPI contracts into reusable components while maintaining auto-completion and validation capabilities. This guide introduces the x-component extension for modular API design
image:
  path: /assets/assets/img/2025-04-10-building-asyncapi-like-lego/cover image.png
---

## Intro

[Kin Lane](https://www.linkedin.com/in/kinlane/) reposted an interesting article about building OpenAPI specs from reusable blocks to save time and increase the reusability of already crafted elements.

Here’s a follow-up addressing one of the key limitations highlighted:

> One major downside is that each file is “just” YAML, so there’s no OpenAPI file validation available when you’re working on the individual elements.

## Solution

I’d like to suggest that the [OpenAPI Initiative](https://www.openapis.org) consider the same approach I’m currently working on for the [AsyncAPI Initiative](https://www.asyncapi.com): using an `x-component` object to store metadata for extracted components.

The idea is simple — while it might be difficult to standardize and officially include this structure in the core specification, it’s easy to implement it through an extension

A simple structure that’s sufficient to describe each of the 21 extractable components outside the API contract, without losing features like auto-completion and validation

```json
{
  "clientName": "transactions-broker",
  "msgVpn": "ProdVPN",
  "bindingVersion": "0.4.0",
  "x-component": {
    "type": "server-binding",
    "bindingType": "solace",
    "version": "3.24.0",
    "specificationVersion": "3.0.0",
    "description": "Solace transactions broker"
  }
}
```
{:file='solace-server-binding.3.24.0.json'}

Now any tool can provide needed information - here is `ServerBinding` for Solace which can be used in any API which is based on AsyncAPI v3

Here is the complete list of components that can be extracted from AsyncAPI contracts:
- Channel
- ChannelBinding
- CorrelationId
- ExternalDocumentation
- Headers
- Message
- MessageBinding
- MessageTrait
- Operation
- OperationBinding
- OperationReply
- OperationReplyAddress
- OperationTrait
- Parameter
- Payload
- Schema
- SecurityScheme
- Server
- ServerBinding
- ServerVariable
- Tag
