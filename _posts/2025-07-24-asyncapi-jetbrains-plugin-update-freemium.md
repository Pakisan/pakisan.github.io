---
title: 'AsyncAPI JetBrains Plugin: Freemium Model & Spectral Linter Update'
description: Discover the new freemium model and Spectral Linter integration in the AsyncAPI JetBrains Plugin. Learn what's changing and how it benefits JetBrains IDE users
date: 2025-07-24 15:24:37 +0400
author: pakisan
categories: [Development, JetBrains IDEs]
tags: [JetBrains Plugin, asyncapi]
head:
  - - meta
    - name: keywords
      content: asyncapi jetbrains plugin, spectral linter integration, jetbrains plugin freemium, asyncapi plugin features, jetbrains ides, asyncapi plugin update, spectral validation jetbrains, asyncapi plugin pricing
  - - link
    - rel: canonical
      href: https://pavelon.dev/posts/2025-07-24-upcoming-jetbrains-plugin-release-1/
  - - meta
    - property: og:title
      content: 'AsyncAPI JetBrains Plugin: Freemium Model & Spectral Linter Update'
  - - meta
    - property: og:description
      content: Discover the new freemium model and Spectral Linter integration in the AsyncAPI JetBrains Plugin. Learn what's changing and how it benefits JetBrains IDE users
  - - meta
    - property: og:type
      content: article
  - - meta
    - property: og:url
      content: https://pavelon.dev/posts/2025-07-24-upcoming-jetbrains-plugin-release-1/
  - - meta
    - name: twitter:title
      content: 'AsyncAPI JetBrains Plugin: Freemium Model & Spectral Linter Update'
  - - meta
    - name: twitter:description
      content: Discover the new freemium model and Spectral Linter integration in the AsyncAPI JetBrains Plugin. Learn what's changing and how it benefits JetBrains IDE users
image:
  path: /assets/assets/img/2025-07-24-upcoming-jetbrains-plugin-release-1/spectral-settings.png
---

# 🚀 Upcoming Changes to the AsyncAPI JetBrains Plugin: Freemium Model & Spectral Integration

Hey folks 👋

I wanted to share some important updates about the future of the AsyncAPI JetBrains Plugin — including a shift to a **freemium model** and the announcement of a brand-new paid feature: **Spectral Linter integration**.

## Why the switch to freemium?

### 💡 The Motivation

Since day one, I’ve been building the AsyncAPI plugin and other developer tools solo — driven by passion, not profit. The goal has always been to support the community with essential tools that make adopting AsyncAPI smoother across different organizations.

But over the past few months, I’ve been thinking: *How can I develop faster, improve quality, and take on bigger projects like a Language Server for AsyncAPI?*

The answer boiled down to two options:

1. Continue relying on donations (which are unpredictable), or
2. Turn the plugin into a sustainable product.

After much thought, I’ve decided to move to a **freemium model**. This will help me estimate revenue more reliably and plan future development better — without burning out.

### 🤝 What stays free?

All core features will **remain free** — everything you need for basic authoring and validation of AsyncAPI specs.

Paid features will build on top of the core experience, offering extended capabilities for those who want more power and convenience.

Here’s a breakdown:

| Feature                                                  | Free | Status       |
|----------------------------------------------------------|------|--------------|
| AsyncAPI spec recognition                                | ✅   | Released     |
| Completion support                                       | ✅   | Released     |
| Spec structure validation                                | ✅   | Released     |
| Local references resolution + completion                 | ✅   | Released     |
| External file references + completion                    | ✅   | Released     |
| Internal/external browser preview                        | ✅   | Released     |
| Native preview with custom FileEditor                    | ✅   | Released     |
|                                                          |      |              |
| **Spectral linter integration**                          | ❌   | Coming Soon  |
| Redocly linter integration                               | ❌   | In Progress  |
| Visual editor (edit via UI)                              | ❌   | In Progress  |
| Integration with registries, test suites, mock tools     | ❌   | Planned      |

> A more detailed and up-to-date roadmap will be published soon on the dedicated [JetBrains Plugin page](https://asyncapi.pavelon.dev/jetbrains-plugin.html). 
There, you'll be able to track upcoming features, planned improvements, and long-term goals for the plugin's development.

---

## 🎯 First Paid Feature: Spectral Linter Integration

The first paid feature coming in the next release is **Spectral Linter integration**.

This feature gives you real-time and manual validation using [Spectral](https://docs.stoplight.io/docs/spectral), helping you catch spec issues early and often.

### 🗂️ Control What Gets Validated

Ever get annoyed by tools validating files you don’t care about? Me too.

That’s why this integration supports a `.spectralignore` file — with the same syntax as `.gitignore`.

Want to skip a folder but include a specific file? Easy:

```text
streetlights/
!streetlights/mqtt.yaml
```
{:file='.spectralignore'}

### ⚙️ Configure Once, Enjoy Always

![Spectral Settings](/assets/assets/img/2025-07-24-upcoming-jetbrains-plugin-release-1/spectral-settings.png){: width="972" height="589" }

You can configure your Spectral ruleset and settings right inside JetBrains IDEs.

### ✅ Validate Your Specs – Automatically or Manually

Whether you're writing or editing your AsyncAPI spec, get instant feedback as you go:

![Receive Feedback](/assets/assets/img/2025-07-24-upcoming-jetbrains-plugin-release-1/spectral-valid.gif){: width="972" height="589" }

![Specification Validation](/assets/assets/img/2025-07-24-upcoming-jetbrains-plugin-release-1/spectral-validate.gif){: width="972" height="589" }

Want to trigger validation manually? You can do that too:

![Specification manual invocation](/assets/assets/img/2025-07-24-upcoming-jetbrains-plugin-release-1/spectral-manual-linter-invocation.gif){: width="972" height="589" }

## ❤️ Thanks for your support

I deeply appreciate everyone who has used, tested, or shared feedback on the plugin. Your support keeps this project alive and moving forward.

This freemium model isn’t about locking things behind a paywall - it’s about sustainability, speed, and building more awesome tools for you.

If you have thoughts, concerns, or feature requests, feel free to [reach out on GitHub](https://github.com/Pakisan/asyncapi-developer-portal/discussions/2) or drop me a message.

Stay tuned for the release — coming soon!
