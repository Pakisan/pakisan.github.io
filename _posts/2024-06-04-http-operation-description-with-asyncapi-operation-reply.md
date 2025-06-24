---
title: Describing HTTP operation with AsyncAPI Operation Reply | Practical Guide
date: 2024-06-04 02:00:00 +0400
author: pakisan
categories: [Specifications, AsyncAPI]
tags: [asyncapi, http]
description: Learn how to describe HTTP operations using AsyncAPI v3 Operation Reply with practical examples. This guide shows how to implement Server-Sent Events (SSE) and HTTP messaging in AsyncAPI specifications
head:
  - - meta
    - name: keywords
      content: asyncapi v3, operation reply, http operations, server-sent events, sse, api specification, event-driven architecture, asyncapi http binding, message broadcasting
  - - meta
    - property: og:title
      content: Describing HTTP operation with AsyncAPI Operation Reply | Practical Guide
  - - meta
    - property: og:description
      content: Learn how to describe HTTP operations using AsyncAPI v3 Operation Reply with practical examples. This guide shows how to implement Server-Sent Events (SSE) and HTTP messaging in AsyncAPI specifications
  - - meta
    - property: og:type
      content: article
  - - meta
    - property: og:url
      content: https://pavelon.dev/posts/http-operation-description-with-asyncapi-operation-reply/
  - - meta
    - name: twitter:title
      content: Describing HTTP operation with AsyncAPI Operation Reply | Practical Guide
  - - meta
    - name: twitter:description
      content: Learn how to describe HTTP operations using AsyncAPI v3 Operation Reply with practical examples. This guide shows how to implement Server-Sent Events (SSE) and HTTP messaging in AsyncAPI specifications.
image:
  path: /assets/assets/img/2024-06-04-http-operation-description-with-asyncapi-operation-reply/cover image.png
---

Few weeks ago I saw interesting question in our Slack.

> Thanks, but I'm looking for v3 examples - where operation reply definition can be used
afaik it's not available in v2

It was about how to describe HTTP operation with [Operation Reply](https://www.asyncapi.com/docs/reference/specification/v3.0.0#operationReplyObject).
In previous note I have already written about describing of SSE application with AsyncAPI.

It's time to show how to adopt those example for AsyncAPI v3

## Contract description

Service which will:
- Broadcast received messages through a Server-Sent Events connection to any subscribed user
- Receive messages for broadcasting trough HTTP Resource `/messages`

## Basic contract structure

Let's start from basic contract structure:

```yaml
asyncapi: '3.0.0'
info:
  title: HTTP operations example
  version: 1.0.0
  description: This example shows how to describe HTTP operations with AsyncAPI v3
```
{: file='http operations example.yaml'}

## Introducing channel

Let's create our channel with address equals to `/messages`.

We will use it for two things:
- Subscribe to stream for receiving of incoming messages
- Send messages for further broadcasting

```yaml
asyncapi: '3.0.0'
info:
  title: HTTP operations example
  version: 1.0.0
  description: This example shows how to describe HTTP operations with AsyncAPI v3
channels:
  messages:
    address: "/messages"
    description: |
      Channel to:
      - subscribe to SSE connection stream to receive incoming messages
      - send messages to broadcast trough SSE
```
{: file='http operations example.yaml'}

## Introducing messages

When channel was defined, it's time to tell which messages can be received or sent.

- `messagesStream` it's a wrapper for our response, which holds important response headers
- `messageToBroadcast` it's a message which user can send for further broadcasting to all subscribed users
- `messageReadyForBroadcasting` it's a message which subscribed users will receive trough SSE connection

```yaml
asyncapi: '3.0.0'
info:
  title: HTTP operations example
  version: 1.0.0
  description: This example shows how to describe HTTP operations with AsyncAPI v3
channels:
  messages:
    address: "/messages"
    description: |
      Channel to:
      - subscribe to SSE connection stream to receive incoming messages
      - send messages to broadcast trough SSE
components:
  messages:
    messagesStream:
      summary: Stream of Messages
      payload:
        $ref: "#/components/messages/messageReadyForBroadcasting/payload"
      bindings:
        http:
          headers:
            properties:
              Content-Type:
                type: string
                enum: ["text/event-stream"]
              X-SSE-Content-Type:
                type: string
                enum: ["application/json"]
              transfer-encoding:
                enum: ["chunked"]
          statusCode: 200
          bindingVersion: 0.3.0
    messageToBroadcast:
      payload:
        type: object
        additionalProperties: false
        description: Message received by application for further broadcasting
        required: [message]
        properties:
          message:
            type: object
            additionalProperties: false
            description: Ordinary text which will be send
            examples:
              - broadcast this message \uD83D\uDE80
    messageReadyForBroadcasting:
      payload:
        type: object
        additionalProperties: false
        description: Broadcasting ready message
        required: [message, receivedAt]
        properties:
          message:
            type: object
            additionalProperties: false
            description: Ordinary text which will be send
            examples:
              - broadcast this message \uD83D\uDE80
          receivedAt:
            type: string
            format: date-time
            description: Date-time when application received this message
            examples:
              - 2023-08-31T15:28:21.283+00:00
```
{: file='http operations example.yaml'}

## Binding channel with messages

In our case our channel allows to `send` and to `receive` messages.

That's why we must bind it with all messages.

```yaml
asyncapi: '3.0.0'
info:
  title: HTTP operations example
  version: 1.0.0
  description: This example shows how to describe HTTP operations with AsyncAPI v3
channels:
  messages:
    address: "/messages"
    description: |
      Channel to:
      - subscribe to SSE connection stream to receive incoming messages
      - send messages to broadcast trough SSE
    messages:
      messageToBroadcast:
        $ref: "#/components/messages/messageToBroadcast"
      messagesStream:
        $ref: "#/components/messages/messagesStream"
components:
  messages:
    messagesStream:
      summary: Stream of Messages
      payload:
        $ref: "#/components/messages/messageReadyForBroadcasting/payload"
      bindings:
        http:
          headers:
            properties:
              Content-Type:
                type: string
                enum: ["text/event-stream"]
              X-SSE-Content-Type:
                type: string
                enum: ["application/json"]
              transfer-encoding:
                enum: ["chunked"]
          statusCode: 200
          bindingVersion: 0.3.0
    messageToBroadcast:
      payload:
        type: object
        additionalProperties: false
        description: Message received by application for further broadcasting
        required: [message]
        properties:
          message:
            type: object
            additionalProperties: false
            description: Ordinary text which will be send
            examples:
              - broadcast this message \uD83D\uDE80
    messageReadyForBroadcasting:
      payload:
        type: object
        additionalProperties: false
        description: Broadcasting ready message
        required: [message, receivedAt]
        properties:
          message:
            type: object
            additionalProperties: false
            description: Ordinary text which will be send
            examples:
              - broadcast this message \uD83D\uDE80
          receivedAt:
            type: string
            format: date-time
            description: Date-time when application received this message
            examples:
              - 2023-08-31T15:28:21.283+00:00
```
{: file='http operations example.yaml'}

## Introducing send operation

To send message for broadcasting to our subscribed user, operation must be created.

This operation must have next properties:
- action is `send`
- channel is `/messages`
- **request schema**: messages is array with only one value - reference to `#/channels/messages/messages/messageToBroadcast`
- HTTP binding which defines that `POST` method will be used
- **response schema**: operation reply definitions where we will define:
  - channel equals to `#/channels/messages`
  - messages is array with only one value - reference to `#/channels/messages/messages/messageToBroadcast`

```yaml
asyncapi: '3.0.0'
info:
  title: HTTP operations example
  version: 1.0.0
  description: This example shows how to describe HTTP operations with AsyncAPI v3
channels:
  messages:
    address: "/messages"
    description: |
      Channel to:
      - subscribe to SSE connection stream to receive incoming messages
      - send messages to broadcast trough SSE
    messages:
      messageToBroadcast:
        $ref: "#/components/messages/messageToBroadcast"
      messagesStream:
        $ref: "#/components/messages/messagesStream"
operations:
  publishMessage:
    action: send
    description: Send message to subscribed users
    channel:
      $ref: "#/channels/messages"
    messages:
      - $ref: "#/channels/messages/messages/messageToBroadcast" # Request schema
    reply:
      channel:
        $ref: "#/channels/messages"
      messages:
        - $ref: "#/channels/messages/messages/messageToBroadcast" # Response schema
    bindings:
      http:
        method: POST
        bindingVersion: 0.3.0
components:
  messages:
    messagesStream:
      summary: Stream of Messages
      payload:
        $ref: "#/components/messages/messageReadyForBroadcasting/payload"
      bindings:
        http:
          headers:
            properties:
              Content-Type:
                type: string
                enum: ["text/event-stream"]
              X-SSE-Content-Type:
                type: string
                enum: ["application/json"]
              transfer-encoding:
                enum: ["chunked"]
          statusCode: 200
          bindingVersion: 0.3.0
    messageToBroadcast:
      payload:
        type: object
        additionalProperties: false
        description: Message received by application for further broadcasting
        required: [message]
        properties:
          message:
            type: object
            additionalProperties: false
            description: Ordinary text which will be send
            examples:
              - broadcast this message \uD83D\uDE80
    messageReadyForBroadcasting:
      payload:
        type: object
        additionalProperties: false
        description: Broadcasting ready message
        required: [message, receivedAt]
        properties:
          message:
            type: object
            additionalProperties: false
            description: Ordinary text which will be send
            examples:
              - broadcast this message \uD83D\uDE80
          receivedAt:
            type: string
            format: date-time
            description: Date-time when application received this message
            examples:
              - 2023-08-31T15:28:21.283+00:00
```
{: file='http operations example.yaml'}

## Introducing receive operation

To subscribe to stream of messages we must create operation with next properties:
- action is `receive`
- channel is `/messages`
- messages is array with only one value - reference to `#/channels/messages/messages/messagesStream`
- HTTP binding which defines that `GET` method will be used

```yaml
asyncapi: '3.0.0'
info:
  title: HTTP operations example
  version: 1.0.0
  description: This example shows how to describe HTTP operations with AsyncAPI v3
channels:
  messages:
    address: "/messages"
    description: |
      Channel to:
      - subscribe to SSE connection stream to receive incoming messages
      - send messages to broadcast trough SSE
    messages:
      messageToBroadcast:
        $ref: "#/components/messages/messageToBroadcast"
      messagesStream:
        $ref: "#/components/messages/messagesStream"
operations:
  subscribesToMessagesStream:
    action: receive
    description: Returns SSE stream of messages
    channel:
      $ref: "#/channels/messages"
    messages:
      - $ref: "#/channels/messages/messages/messagesStream"
    bindings:
      http:
        method: GET
        bindingVersion: 0.3.0
  publishMessage:
    action: send
    description: Send message to subscribed users
    channel:
      $ref: "#/channels/messages"
    messages:
      - $ref: "#/channels/messages/messages/messageToBroadcast"
    reply:
      channel:
        $ref: "#/channels/messages"
      messages:
        - $ref: "#/channels/messages/messages/messageToBroadcast"
    bindings:
      http:
        method: POST
        bindingVersion: 0.3.0
components:
  messages:
    messagesStream:
      summary: Stream of Messages
      payload:
        $ref: "#/components/messages/messageReadyForBroadcasting/payload"
      bindings:
        http:
          headers:
            properties:
              Content-Type:
                type: string
                enum: ["text/event-stream"]
              X-SSE-Content-Type:
                type: string
                enum: ["application/json"]
              transfer-encoding:
                enum: ["chunked"]
          statusCode: 200
          bindingVersion: 0.3.0
    messageToBroadcast:
      payload:
        type: object
        additionalProperties: false
        description: Message received by application for further broadcasting
        required: [message]
        properties:
          message:
            type: object
            additionalProperties: false
            description: Ordinary text which will be send
            examples:
              - broadcast this message \uD83D\uDE80
    messageReadyForBroadcasting:
      payload:
        type: object
        additionalProperties: false
        description: Broadcasting ready message
        required: [message, receivedAt]
        properties:
          message:
            type: object
            additionalProperties: false
            description: Ordinary text which will be send
            examples:
              - broadcast this message \uD83D\uDE80
          receivedAt:
            type: string
            format: date-time
            description: Date-time when application received this message
            examples:
              - 2023-08-31T15:28:21.283+00:00
```
{: file='http operations example.yaml'}
