---
title: 'AsyncAPI JetBrains Plugin: Digest #2 - Smarter References, YAML & JSON Editing, Project Overview'
description: Second feature digest of the AsyncAPI JetBrains Plugin. Everything since Digest
date: 2026-09-08 13:49:24 +0400
author: pakisan
categories: [Development, JetBrains IDEs]
tags: [JetBrains Plugin, AsyncAPI, Spring Boot, Apache Kafka]
head:
  - - meta
    - name: keywords
      content: asyncapi jetbrains plugin, asyncapi ide, jetbrains plugin, asyncapi reference resolving, asyncapi yaml editor, asyncapi project overview, spring boot asyncapi, apache kafka asyncapi
  - - link
    - rel: canonical
      href: https://pavelon.dev/posts/asyncapi-jetbrains-plugin-digest-2/
  - - meta
    - property: og:title
      content: 'AsyncAPI JetBrains Plugin: Digest #2 - Smarter References, YAML & JSON Editing, Project Overview'
  - - meta
    - property: og:description
      content: Second digest of the AsyncAPI JetBrains Plugin. Rebuilt reference resolving, visual editing for YAML and JSON, standalone components, and Spring messaging discovery.
  - - meta
    - property: og:type
      content: article
  - - meta
    - property: og:url
      content: https://pavelon.dev/posts/asyncapi-jetbrains-plugin-digest-2/
  - - meta
    - name: twitter:title
      content: 'AsyncAPI JetBrains Plugin: Digest #2 - References, Editing & Project Overview'
  - - meta
    - name: twitter:description
      content: New digest of the AsyncAPI JetBrains Plugin with a rebuilt reference resolving engine, YAML/JSON visual editing, and Project Overview for Spring.
image:
  path: /assets/assets/img/2026-02-09-asyncapi-jetbrains-plugin-digest-1/cover.jpg
---

# 🚀 AsyncAPI JetBrains Plugin - Digest #2

Hey folks 👋

Welcome to the second **[AsyncAPI JetBrains Plugin](https://plugins.jetbrains.com/plugin/15673-asyncapi) Digest**.

[Digest #1](https://pavelon.dev/posts/asyncapi-jetbrains-plugin-digest-1/) stopped around version **4.3.0**. A lot has landed since then, so this one is bigger. It is split into two tracks:

- **For AsyncAPI authors** - references, the YAML/JSON visual editor, standalone components, preview, and validation.
- **For engineers working with Spring applications** - Project Overview and messaging discovery.

Let's jump in 👇

---

## ✍️ For AsyncAPI authors

### 🔗 Reference resolving - back, and rebuilt

Reference resolving was **temporarily removed in 4.3.0** while the new UI landed, **returned in 4.5.0**, and has now been **completely rebuilt in 4.12.0** as a single engine shared by the editor and the preview.

What you get today:

- **Every kind of `$ref`** - local pointers (`#/components/messages/UserSignedUp`), file references (relative, absolute, bare filename, Windows paths, `file:` URIs), and remote `http` / `https` references - resolved the same way in the editor and in the preview.
- **File references are much easier to write.** As you type a reference to another file, completion lists the contents of the **current folder**, so you can find and pick the right `.json` or `.yaml` document without remembering its path. Rename a referenced file and every `$ref` to it updates automatically.
- **JSON Pointers that go anywhere.** After the `#`, completion offers the elements *inside* the target document - a local file **or a remote URL** - so you can point straight at the exact node you need, for example one `server` or one `message`. *Go to Declaration* (`Ctrl/Cmd+B`) follows the pointer into that document's own content and puts the caret on the element.
- **Reference chains, cross-language references, Avro, cycles.** A `$ref` that points at another `$ref` is followed to the end, a JSON document can reference a YAML one (and the reverse), `.avsc` schemas are parsed, and reference cycles are detected and reported instead of hanging.
- **A dedicated inspection** names each problem - unreachable document, pointer that finds nothing, unsupported fragment, cycle, host awaiting a decision - with its own quick fix.

![Resolved reference in the AsyncAPI Explorer](/assets/assets/img/2026-09-08-asyncapi-jetbrains-plugin-digest-2/reference-resolved.png){: width="972" height="589" }

**Remote hosts stay under your control.** A reference to an `http` or `https` address is **never fetched until you allow its host**. You answer *Allow* or *Deny* once, the decision is stored per project, and a redirect onto a new host asks again.

![Request to allow an unknown host](/assets/assets/img/2026-09-08-asyncapi-jetbrains-plugin-digest-2/allow-unknown-host.png){: width="972" height="589" }

Every answer, plus the read timeout, maximum document size, and maximum reference depth, lives in *Settings → Tools → AsyncAPI → Remote References*.

![Remote References settings](/assets/assets/img/2026-09-08-asyncapi-jetbrains-plugin-digest-2/remote-references-settings.png){: width="972" height="589" }

**Free vs licensed.** Local and file references - resolving, current-folder completion, JSON Pointer navigation - are free. Remote references are free from **one** approved host, and denying hosts is unlimited. Allowing **more than one** host, and routing resolution through an **HTTP proxy**, are licensed features.

---

### 🖊️ Visual editing for JSON *and* YAML

The UI editor started as a JSON-only feature preview in **4.3.0**. Since then:

- **4.11.0 adds YAML editing** - the visual editor now works on YAML documents too, not just JSON.
- The JSON/YAML mutation engine was **rebuilt** so edits on deeply nested fields are reliable, and missing parent objects are created automatically when you add a nested element.
- Editor and UI stay **bidirectionally in sync** - switch between text and form without losing your place.

Enabling edit mode is a licensed feature; **read-only exploration of a contract - the tree, navigation, and detail views - is free.**

---

### 🧩 Standalone components as first-class files

You no longer have to keep everything in one giant document.

- **4.10.0 adds "New …" actions** for AsyncAPI v3 building blocks - Server, Server Variable, Channel, Operation, Operation Reply, Operation Reply Address, and Message - pre-filled for **19 protocols**: Amazon SNS, Amazon SQS, AMQP 0-9-1, AMQP 1.0, Anypoint MQ, Apache Kafka, Apache Pulsar, Google Cloud Pub/Sub, HTTP, IBM MQ, JMS, Mercure, MQTT, MQTT v5.0, NATS, Redis, Solace, STOMP, WebSockets.
- Standalone **Server, Channel, Operation, Message** files (and their traits, correlation IDs, replies, and parameters) are **validated, autocompleted, and previewed on their own**, not only as part of a full contract.

Creating, validating, completing, and previewing components is a licensed feature.

---

### 👁️ Preview improvements

- **4.7.0** - preview AsyncAPI **3.1.0** documents.
- **4.6.0** - preview AsyncAPI **2.6.0** documents.
- **4.5.0** - the native preview panel became the **default**; the legacy React-based view was retired.

---

### ✅ Validation & schema accuracy

- **4.10.0** - validation and completion now run against dedicated, up-to-date AsyncAPI JSON Schemas for **2.x and 3.x**, covering both full contracts and standalone components.
- The bundled JetBrains JSON Schema validation, which broke in the 2026.* IDEs, is **disabled** - the plugin provides validation instead.
- **4.5.1** - schema fixes: `$ref` values may contain `{` and `}` (parametrized channel references), and unknown properties are rejected except `x-` extensions.

---

### 🌍 Localization

**4.5.0** refreshed the English strings and added **German** and **Simplified Chinese** translations.

---

## 🌱 For engineers working with Spring applications

### 🔎 Project Overview

**4.8.0** introduced the **Project Overview** tool window: every AsyncAPI **document**, **component**, and messaging **endpoint** in your project, in one place.

- **Multi-module aware** - correct broker and endpoint attribution for Gradle and Maven source sets.
- **Quick document access** - open any AsyncAPI spec or standalone component, filtered by AsyncAPI version and component type.
- Built to stay fast on large repositories with hundreds of listeners and documents.

Project Overview is a licensed feature.

---

### 📡 Messaging discovery - growing annotation coverage

Instead of hunting for annotations by hand, let the plugin map your event-driven architecture:

- **4.8.0**
  - **Apache Kafka** `@KafkaListener` (class- and method-level) and **RabbitMQ** `@RabbitListener`.
- **4.9.0**
  - **Amazon SNS** (`@NotificationMessageMapping`, `@NotificationSubscriptionMapping`, `@NotificationUnsubscribeConfirmationMapping`)
  - **Amazon SQS** (`@SqsListener`, `@SqsHandler`)
  - **Apache Kafka** `@KafkaHandler`
  - **Apache Pulsar** `@PulsarListener`
  - **JMS** `@JmsListener`
  - and **STOMP** (`@MessageMapping`, `@SubscribeMapping`).

Every detected endpoint links straight to its method or class in the editor, and the list is searchable by class, method, or Javadoc and filterable by broker.

---

### ⚠️ Removed - code-to-AsyncAPI generation

The earlier **"generate an AsyncAPI contract from your Spring / Kafka code"** feature - covered back in Digest #1 - **has been removed**.

Project Overview replaces it for now with **discovery and navigation**: it shows you what your code publishes and consumes, and takes you to it, without writing a spec on your behalf. Generation is expected to return later as part of the framework and MCP work below.

---

## 🧭 What's next

No dates, just direction:

- **MCP server** - connect AI agents to the plugin so they can scan your project, validate specs, and generate AsyncAPI documents through open, standard tooling.
- **More Spring coverage** and **deeper broker integrations** - additional annotations, configuration sources, and protocols in Project Overview.

Follow the digests to see these land.

---

## ❤️ Thanks for Your Support

A huge thank you to everyone who has tried the plugin, reported issues, shared feedback, or spread the word.
Your support directly shapes where this project goes next.

Have ideas, questions, or feature requests? Drop me a message - I'm always happy to chat.
