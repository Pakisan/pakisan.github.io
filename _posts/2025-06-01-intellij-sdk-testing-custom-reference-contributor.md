---
title: 'Behind the Scenes: Reference Completion in the AsyncAPI Plugin for JetBrains IDEs'
description: How to test own implementation of reference contributor in IntelliJ SDK
date: 2025-06-01 22:16:44 +0400
author: pakisan
categories: [Development, JetBrains IDEs]
tags: [IntelliJ SDK]
image:
  path: /assets/assets/img/asyncapi-plugin-for-jetbrains-ide.png
---

Today I want to start sharing more about how my [AsyncAPI Specification](https://asyncapi.com) plugin for JetBrains IDEs works under the hood. The goal is to pass along some of the knowledge I’ve gathered and hopefully inspire or help others in the community build awesome plugins

**Topic of the Day:** Reference Contributors and Auto-Completion

In this post, we’ll explore how the plugin handles custom reference contributors and provides auto-completion for references in JSON-based AsyncAPI documents. I’ll also show what this looks like in practice and how you can test it effectively

## Implementation

Rather than dumping large code blocks into this article, I’ll link to relevant source files for further study

These classes relate specifically to JSON:
- [AsyncAPISpecificationReferenceContributor](https://github.com/asyncapi/jasyncapi-idea-plugin/blob/master/src/main/kotlin/com/asyncapi/plugin/idea/extensions/psi/reference/contributor/json/AsyncAPISpecificationReferenceContributor.kt): This class is registered as a PSI Reference Contributor for files with the JSON language. It tells the IDE how references should be handled and which strategies are available for resolving them

Providers for different kinds of references:
- [AsyncAPIFileReferenceProvider](https://github.com/asyncapi/jasyncapi-idea-plugin/blob/master/src/main/kotlin/com/asyncapi/plugin/idea/extensions/psi/reference/provider/json/AsyncAPIFileReferenceProvider.kt): Handles references to external files
- [AsyncAPILocalReferenceProvider](https://github.com/asyncapi/jasyncapi-idea-plugin/blob/master/src/main/kotlin/com/asyncapi/plugin/idea/extensions/psi/reference/provider/json/AsyncAPILocalReferenceProvider.kt): Handles references within the same file

Each provider returns a specific reference type:
- [AsyncAPIFileReference](https://github.com/asyncapi/jasyncapi-idea-plugin/blob/master/src/main/kotlin/com/asyncapi/plugin/idea/extensions/psi/reference/AsyncAPIFileReference.kt): Used for file references
- [AsyncAPILocalReference](https://github.com/asyncapi/jasyncapi-idea-plugin/blob/master/src/main/kotlin/com/asyncapi/plugin/idea/extensions/psi/reference/AsyncAPILocalReference.kt): Used for local references

Finally, the [JsonFileVariantsProvider](https://github.com/asyncapi/jasyncapi-idea-plugin/blob/master/src/main/kotlin/com/asyncapi/plugin/idea/extensions/psi/reference/JsonFileVariantsProvider.kt) resolves and inspects references, determines if a file should be parsed, and prepares completion suggestions based on the caret position inside a $ref node

## Testing

To ensure everything works as expected, we write a test that extends JetBrains' [BasePlatformTestCase](https://github.com/JetBrains/intellij-community/blob/master/platform/testFramework/src/com/intellij/testFramework/fixtures/BasePlatformTestCase.java)

```kotlin
class JsonReferenceVariantsTest: BasePlatformTestCase() {
  
}
```

You’ll need to specify the path to the test data:

```kotlin
class JsonReferenceVariantsTest: BasePlatformTestCase() {

  override fun getTestDataPath(): String = "src/test/testData/json/reference/completion/3.0.0"

}
```

Here’s what you’ll find in that path:

A sample AsyncAPI spec with a `$ref` containing a caret position:
```json
{
  "asyncapi": "3.0.0",
  "info": {
    "title": "Account Service",
    "version": "1.0.0",
    "description": "This service is in charge of processing user signups :rocket:"
  },
  "channels": {
    "userSignedup": {
      "address": "user/signedup",
      "messages": {
        "userSignedupMessage": {
          "$ref": "./ref.json#/r<caret>"
        }
      }
    }
  }
}
```

The file being referenced, containing a simple object:
```json
{
  "reference": "reference value"
}
```
{:file='ref.json'}

### Expected Behavior

When the user types` #/r`, the plugin should parse `ref.json` and suggest any top-level keys that start with `r`. In this case, the expected completion is:

```text
#/reference
```

### The Test

Here’s the actual test code:

```kotlin
fun `test referenced file`() {
  val referenceAtCaret = myFixture.getReferenceAtCaretPositionWithAssertion("reference.json", "ref.json")
  val lookupElement = referenceAtCaret.variants.first() as LookupElement
  assertEquals("./ref.json#/reference", lookupElement.lookupString)
}
```

Reference - [Simple Language Plugin](https://github.com/JetBrains/intellij-sdk-code-samples/tree/master/simple_language_plugin) example

## Final Thoughts

Custom reference handling is a core feature of a developer-friendly IDE experience. Building intelligent auto-completion and reference resolution helps developers navigate and write AsyncAPI specifications more efficiently

This is just the beginning of the series. Stay tuned for deeper dives into the internals of the plugin — and feel free to explore the [AsyncAPI Plugin on GitHub](https://github.com/asyncapi/jasyncapi-idea-plugin) if you’re curious or want to contribute!
