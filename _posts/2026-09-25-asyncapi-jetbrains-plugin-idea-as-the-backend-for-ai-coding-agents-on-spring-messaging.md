---
title: 'AsyncAPI JetBrains Plugin: IDEA as the Backend for AI Coding Agents on Spring Messaging'
description: A new CLI ↔ IDE interface is coming, and the AsyncAPI JetBrains plugin is already positioned for it — turning IntelliJ IDEA into a backend AI coding agents can query for accurate, code-level facts about Spring Kafka, Pulsar, SQS, SNS, JMS and STOMP messaging, with MCP support on the way
date: 2026-09-25 12:31:45 +0400
author: pakisan
categories: [Development, JetBrains IDEs]
tags: [JetBrains Plugin, AsyncAPI, Spring Boot, AI, MCP, Agents]
head:
  - - meta
    - name: keywords
      content: ai coding agent jetbrains, mcp server intellij idea, ide cli interface, agent backend spring kafka, llm agent intellij, asyncapi jetbrains plugin, spring kafka intellij, apache pulsar spring, amazon sns spring boot, amazon sqs spring boot, jms listener intellij, stomp websocket spring, kafkalistener inspection, duplicate consumer group kafka, asyncapi export spring boot
  - - link
    - rel: canonical
      href: https://pavelon.dev/posts/asyncapi-jetbrains-plugin-idea-as-the-backend-for-ai-coding-agents-on-spring-messaging/
  - - meta
    - property: og:title
      content: 'AsyncAPI JetBrains Plugin: IDEA as the Backend for AI Coding Agents on Spring Messaging'
  - - meta
    - property: og:description
      content: A new CLI ↔ IDE interface is coming, and the AsyncAPI JetBrains plugin is already positioned for it — code-level AsyncAPI export, cross-protocol topology tracing, and inspections an agent can rely on before the app even starts
  - - meta
    - property: og:type
      content: article
  - - meta
    - property: og:url
      content: https://pavelon.dev/posts/asyncapi-jetbrains-plugin-idea-as-the-backend-for-ai-coding-agents-on-spring-messaging/
  - - meta
    - name: twitter:title
      content: 'AsyncAPI JetBrains Plugin: IDEA as the Backend for AI Coding Agents'
  - - meta
    - name: twitter:description
      content: IntelliJ IDEA becomes a backend AI coding agents can query for accurate facts about Spring Kafka, Pulsar, SQS, SNS, JMS and STOMP messaging — with MCP support coming next
---

# How the AsyncAPI Plugin preparing IDEA as a Backend for AI Coding Agents

A few days ago, while scrolling through a newsletter and working on this release, I saw an interesting [post](https://www.linkedin.com/feed/update/urn:li:activity:7506721414738116608/) from [Aleksey Stukalov](https://www.linkedin.com/in/alekseystukalov) about potential requirements for a new IDE ↔ CLI interface and it sounded very familiar

Every new release of the [AsyncAPI plugin](https://plugins.jetbrains.com/plugin/15673-asyncapi) chases two goals: gaining influence, and bringing the right tools inside JetBrains IDEA

Spring messaging development inside IDEA gets sweeter, and at the same time it simplifies AsyncAPI Initiative specification adoption. Even Spring Boot engineers who haven't heard of our specification, or who've ignored it while stuck writing Confluence pages, Markdown docs, or contorted OpenAPI documents to describe Kafka, Pulsar, and message-forwarding rules that were never meant to fit there, may start paying attention and become contributors

## More Than a Specification: Source-Code Analysis Inside IDEA

New release [4.9.5](https://plugins.jetbrains.com/plugin/15673-asyncapi/versions/stable/1179831) shows AsyncAPI is more than a specification: it can extend your or your team's current capabilities through source-code inspection and analysis, right inside IDEA
It also lays the foundation for extending those capabilities to agents, with IDEA as the backend:
- Export your API as an AsyncAPI document through code analysis — for further use or inspection, no compiled or running application required
- Find nested send operations and view a detailed topology of receivers and senders
- Track message forwarding across protocols, so integrations stay visible for analysis — and when you export an API with cross-protocol forwarding, your document captures it too, keeping your documentation up to date automatically
- Get inspections that warn you about real issues before they ship: a duplicated groupId on the same topic, or an empty listener that stops your application from starting

## Why an AI Coding Agent Needs an IDE Backend for Spring Kafka and Messaging Code

Give an agent access to what's already implemented here, and it can work with a Spring application far more accurately, with a much lower chance of a bad change. It can see exactly which method in which class implements a given listener, and whether that conflicts with a listener already sitting somewhere else in the application

It can walk straight to a cross-protocol forward and answer "where else does this message end up" instead of guessing at undefined behavior during manual analysis — or worse, having an agent operate with none of these hints at all

Inspections already predict that a misused annotation means the application won't even start, which removes a whole unneeded hop: no booting the app, no collecting a stack trace, no handing it to an LLM just to learn that an empty `@KafkaListener` was the reason

## What's Next: MCP Commands Inside IDEA

They're only available inside IDEA for now. What matters is that these problems are already solved there — and in the upcoming releases, this same functionality moves to MCP commands inside IDEA, so your agent of choice gets the same evidence to reason with

## Available Today on Community and Ultimate

These features work the same whether you're on IntelliJ IDEA Community or Ultimate

Check out the release notes and give it a try [4.9.5](https://plugins.jetbrains.com/plugin/15673-asyncapi/versions/stable/1179831)
