---
title: 'AsyncAPI JetBrains Plugin: Digest #1 - Smarter UI & Better Kafka Support'
description: First feature digest of the AsyncAPI JetBrains Plugin. Explore the new UI improvements, bidirectional editor, and enhanced AsyncAPI generation from Apache Kafka and Spring-based projects
date: 2025-07-24 15:24:37 +0400
author: pakisan
categories: [Development, JetBrains IDEs]
tags: [JetBrains Plugin, AsyncAPI, Spring Boot, Apache Kafka]
head:
  - - meta
    - name: keywords
      content: asyncapi jetbrains plugin, asyncapi ide, jetbrains plugin, spring boot asyncapi, apache kafka asyncapi
  - - link
    - rel: canonical
      href: https://pavelon.dev/posts/asyncapi-jetbrains-plugin-v4/
  - - meta
    - property: og:title
      content: 'AsyncAPI JetBrains Plugin: Digest #1 - UI Updates & Kafka Improvements'
  - - meta
    - property: og:description
      content: First digest of the AsyncAPI JetBrains Plugin. Discover UI enhancements, UI-based editing, and improved AsyncAPI generation from Apache Kafka.
  - - meta
    - property: og:type
      content: article
  - - meta
    - property: og:url
      content: https://pavelon.dev/posts/asyncapi-jetbrains-plugin-v4/
  - - meta
    - name: twitter:title
      content: 'AsyncAPI JetBrains Plugin: Digest #1 - UI & Kafka Updates'
  - - meta
    - name: twitter:description
      content: New digest of the AsyncAPI JetBrains Plugin with UI improvements and enhanced Kafka-based AsyncAPI generation.
image:
  path: /assets/assets/img/2026-02-09-asyncapi-jetbrains-plugin-digest-1/cover.jpg
---

# 🚀 AsyncAPI JetBrains Plugin - Digest #1

Hey folks 👋

Welcome to the very first **[AsyncAPI JetBrains Plugin](https://plugins.jetbrains.com/plugin/15673-asyncapi) Digest**.  
This series highlights what’s new, what’s improved, and what’s coming - for both **new users** getting started and **existing users** following the evolution of the plugin.

Let’s jump in 👇

---

## UI

### ✨ Added - Bindings Rendering

All supported AsyncAPI bindings are now fully rendered in the UI, making it much easier to explore and understand protocol-specific details directly inside the IDE.

---

### 🎨 Added - Colored Tree Nodes

The AsyncAPI Explorer tree is now color-coded by node type.  
Servers, channels, operations, and components each have their own visual style, making navigation faster and more intuitive - especially for larger specs.

![Colored Tree Nodes](/assets/assets/img/2026-02-09-asyncapi-jetbrains-plugin-digest-1/new-ui-view.png){: width="972" height="589" }


---

### ✏️ Added - Edit Through UI

Starting with version **4.3.0**, the plugin introduces a **feature preview** of UI-based editing.

You can now:
- Edit AsyncAPI documents directly through the UI
- Enjoy **bi-directional synchronization** between the text editor and the UI
- Switch seamlessly between YAML/JSON and visual editing without losing context

This is a big step toward a fully visual AsyncAPI editing experience.

![Edit Through UI](/assets/assets/img/2026-02-09-asyncapi-jetbrains-plugin-digest-1/new-ui-edit.gif){: width="972" height="589" }

---

### 🧠 Fixed - Selected Node State

Good news for multitaskers!  
The selected node in the AsyncAPI Explorer is now preserved when the document changes in the text editor - no more losing your place after a reload.

Additional UI fixes:
- Improved server rendering (adaptive groups replaced with a cleaner vertical layout)
- Fixed rendering glitches in schema editors

---

### 🔄 Changed - UI Layout Improvements

Several views were redesigned to improve readability and information grouping:
- Channel message traits view
- Channel message examples view
- Channel message details view

The goal here is simple: **less scrolling, better overview, faster understanding**.

---

## AsyncAPI Generation from Apache Kafka

AsyncAPI generation for **Apache Kafka** in Spring based applications has been significantly improved.

Previously, the plugin could generate specs only from classes annotated with:
- `@KafkaListener`
- `@KafkaListeners`

In the latest version, generation is more flexible and accurate. You can now register channels from:
- `@KafkaListener` annotated **methods**
- `@KafkaListeners` annotated **methods**

In addition, the plugin can now detect and register **`send` operations** based on `KafkaTemplate` usage - giving you a more complete picture of your async interactions.

👉 For more details, check out the [documentation](https://asyncapi.pavelon.dev/jetbrains-plugin/frameworks/).

---

## ❤️ Thanks for Your Support

A huge thank you to everyone who has tried the plugin, reported issues, shared feedback, or spread the word.  
Your support directly shapes where this project goes next.

Have ideas, questions, or feature requests? Drop me a message - I’m always happy to chat.
