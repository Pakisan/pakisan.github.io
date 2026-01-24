---
title: 'AsyncAPI JetBrains Plugin: 4.0.0'
description: Discover the new major version - AsyncAPI generation from Spring-based projects
date: 2025-07-24 15:24:37 +0400
author: pakisan
categories: [Development, JetBrains IDEs]
tags: [JetBrains Plugin, asyncapi]
head:
  - - meta
    - name: keywords
      content: asyncapi jetbrains plugin, jetbrains plugin, spring boot, apache kafka
  - - link
    - rel: canonical
      href: https://pavelon.dev/posts/asyncapi-jetbrains-plugin-v4/
  - - meta
    - property: og:title
      content: 'AsyncAPI JetBrains Plugin: AsyncAPI generation from Spring-based projects'
  - - meta
    - property: og:description
      content: Discover the new major version of the AsyncAPI JetBrains Plugin - AsyncAPI generation from Spring-based projects
  - - meta
    - property: og:type
      content: article
  - - meta
    - property: og:url
      content: https://pavelon.dev/posts/asyncapi-jetbrains-plugin-v4/
  - - meta
    - name: twitter:title
      content: 'AsyncAPI JetBrains Plugin: AsyncAPI generation from Spring-based projects'
  - - meta
    - name: twitter:description
      content: Discover the new major version of the AsyncAPI JetBrains Plugin - AsyncAPI generation from Spring-based projects
image:
  path: /assets/assets/img/2026-01-26-asyncapi-jetbrains-plugin-v4/cover.png
---

# 🚀 AsyncAPI JetBrains Plugin: 4.0.0

Hey folks 👋

I want to introduce upcoming changes in a new major version of the AsyncAPI JetBrains Plugin

## Switching to IntelliJ IDEA

Most of the current users are using IntelliJ IDEA, to work with AsyncAPI documents. That's why I decided to switch to IntelliJ IDEA as a base for the plugin.

This change allowed me to introduce a new feature - AsyncAPI generation from Spring-based projects

## AsyncAPI generation from Spring-based projects

The plugin now supports generating AsyncAPI documents from Spring-based projects. For now, it's available only for Java-based projects which are using Apache Kafka.

Idea is straightforward: you open your project and click the "Scan" button. After that, the plugin will scan your project and generate an AsyncAPI document based on the Kafka topics and consumers

![Generation](/assets/assets/img/2026-01-26-asyncapi-jetbrains-plugin-v4/spring-boot-apache-kafka-beta.gif){: width="972" height="589" }

More details about this change in the [Documentation](https://asyncapi.pavelon.dev/jetbrains-plugin/frameworks/)

## ❤️ Thanks for your support

I deeply appreciate everyone who has used, tested, or shared feedback on the plugin. Your support keeps this project alive and moving forward.

If you have thoughts, concerns, or feature requests, feel free to [reach out on GitHub](https://github.com/Pakisan/asyncapi-developer-portal/discussions/2) or drop me a message.
