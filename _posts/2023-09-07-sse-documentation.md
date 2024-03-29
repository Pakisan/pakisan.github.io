---
title: SSE documentation
date: 2023-09-07 03:55:00 +0400
author: pakisan
categories: [Specifications]
tags: [asyncapi, openapi, sse]
mermaid: true
---

Most interesting moment for me is to compare OpenAPI specification with AsyncAPI specification in case of:

- re-using of common parts
- describing of SSE application

and compare results

# What will we document

Service which will broadcast received messages through an SSE connection to any subscribed user

```mermaid
sequenceDiagram
  actor User
  participant App
  participant Sse emitter

  User->>App: Send message
  activate App
  Note over App,User: POST /messages HTTP/1.1<br/>Host: localhost:8080<br/>Content-Type: application/json<br/>Content-Length: 46<br/><br/>{"message": "broadcast this message :rocket:"}
  activate Sse emitter
  App--)Sse emitter: pass message
  App-->>User: 
  Note over App,User: HTTP/1.1 200 OK
  deactivate App

  User->>App: Request stream
  Note over App,User: GET /messages HTTP/1.1<br/>Host: localhost:8080
  activate App
  App--)Sse emitter: subscribe
  App-->>User:  
  Note over App,User: HTTP/1.1 200 OK<br/>Content-Type: text/event-stream<br/>X-SSE-Content-Type: application/json<br/>transfer-encoding: chunked
  Sse emitter-)User: broadcast message
  deactivate App
  deactivate Sse emitter
```

# Documentation

## Common part

Let's collect all required schemas in one file:

`schemas.json`
```json
{
  "schemas": {
    "Message": {
      "type": "object",
      "additionalProperties": false,
      "required": [
        "message",
        "receivedAt"
      ],
      "properties": {
        "message": {
          "type": "string",
          "example": "broadcast this message :rocket:"
        },
        "receivedAt": {
          "type": "string",
          "format": "date-time",
          "example": "2023-08-31T15:28:21.283+00:00",
          "description": "Date-time when application received this message"
        }
      }
    },
    "MessageToBroadcast": {
      "type": "object",
      "additionalProperties": false,
      "required": [
        "message"
      ],
      "properties": {
        "message": {
          "type": "string",
          "example": "broadcast this message :rocket:",
          "description": "Ordinary text which will be send"
        }
      }
    }
  }
}
```

## OpenAPI

### Broadcast messages

Just basic declaration, nothing special at all

```json
{
  "openapi": "3.0.3",
  "info": {
    "version": "1.0.0",
    "title": "Messages API",
    "description": "Broadcasts received messages through an SSE connection to any subscribed user"
  },
  "servers": [
    {
      "url": "http://localhost:8080"
    }
  ],
  "paths": {
    "/messages": {
      "post": {
        "summary": "Broadcast message",
        "operationId": "broadcastMessage",
        "description": "Send message to broadcast to any subscribed user",
        "tags": [
          "messages"
        ],
        "requestBody": {
          "description": "Message to broadcast",
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "$ref": "../schemas.json#/schemas/MessageToBroadcast"
              },
              "example": "broadcast this message :rocket:"
            }
          }
        },
        "responses": {
          "200": {
            "description": "message received"
          }
        }
      }
    }
  }
}
```

### Subscribe to messages stream

More complicated part. Unfortunately, community still figuring out how to describe events by OpenAPI in the right manner.

We can use this [GitHub issue](https://github.com/OAI/OpenAPI-Specification/issues/396) as a reference

```json
{
  "openapi": "3.0.3",
  "info": {
    "version": "1.0.0",
    "title": "Messages stream API",
    "description": "Broadcasts received messages through an SSE connection to any subscribed user"
  },
  "servers": [
    {
      "url": "http://localhost:8080"
    }
  ],
  "paths": {
    "/messages": {
      "get": {
        "summary": "Subscribe to stream of messages",
        "operationId": "messagesStreamSubscribe",
        "description": "Receive all incoming messages",
        "tags": [
          "messages"
        ],
        "responses": {
          "200": {
            "description": "Stream of messages",
            "headers": {
              "X-SSE-Content-Type": {
                "schema": {
                  "type": "string",
                  "enum": ["application/json"]
                }
              },
              "transfer-encoding": {
                "schema": {
                  "type": "string",
                  "enum": ["chunked"]
                }
              }
            },
            "content": {
              "text/event-stream": {
                "schema": {
                  "$ref": "#/components/schemas/MessagesStream"
                }
              }
            }
          }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "MessagesStream": {
        "type": "array",
        "format": "event-stream",
        "items": {
          "$ref": "../schemas.json#/schemas/Message"
        }
      }
    }
  }
}
```

### Pub + Sub

```json
{
  "openapi": "3.0.3",
  "info": {
    "version": "1.0.0",
    "title": "Messages API",
    "description": "Broadcasts received messages through an SSE connection to any subscribed user"
  },
  "servers": [
    {
      "url": "http://localhost:8080"
    }
  ],
  "paths": {
    "/messages": {
      "get": {
        "summary": "Subscribe to stream of messages",
        "operationId": "messagesStreamSubscribe",
        "description": "Receive all incoming messages",
        "tags": [
          "messages"
        ],
        "responses": {
          "200": {
            "description": "Stream of messages",
            "headers": {
              "X-SSE-Content-Type": {
                "schema": {
                  "type": "string",
                  "enum": ["application/json"]
                }
              },
              "transfer-encoding": {
                "schema": {
                  "type": "string",
                  "enum": ["chunked"]
                }
              }
            },
            "content": {
              "text/event-stream": {
                "schema": {
                  "$ref": "#/components/schemas/MessagesStream"
                }
              }
            }
          }
        }
      },
      "post": {
        "summary": "Broadcast message",
        "operationId": "broadcastMessage",
        "description": "Send message to broadcast to any subscribed user",
        "tags": [
          "messages"
        ],
        "requestBody": {
          "description": "Message to broadcast",
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "$ref": "../schemas.json#/schemas/MessageToBroadcast"
              },
              "example": "broadcast this message :rocket:"
            }
          }
        },
        "responses": {
          "200": {
            "description": "message received"
          }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "MessagesStream": {
        "type": "array",
        "format": "event-stream",
        "items": {
          "$ref": "../schemas.json#/schemas/Message"
        }
      }
    }
  }
}
```

## AsyncAPI

### Broadcast messages

```json
{
  "asyncapi": "2.6.0",
  "info": {
    "title": "Messages stream API",
    "description": "Broadcasts received messages through an SSE connection to any subscribed user",
    "version": "1.0.0"
  },
  "servers": {
    "dev": {
      "url": "http://localhost:8080",
      "protocol": "http"
    }
  },
  "channels": {
    "/messages": {
      "description": "Broadcast message",
      "publish": {
        "description": "Send message to broadcast to any subscribed user",
        "message": {
          "bindings": {
            "http": {
              "headers": {
                "type": "object",
                "additionalProperties": false,
                "required": [
                  "Content-Type"
                ],
                "properties": {
                  "Content-Type": {
                    "type": "string",
                    "enum": ["application/json"]
                  }
                }
              }
            }
          },
          "$ref": "#/components/messages/MessageToBroadcast"
        },
        "bindings": {
          "http": {
            "type": "request",
            "method": "POST"
          }
        }
      }
    }
  },
  "components": {
    "messages": {
      "MessageToBroadcast": {
        "payload": {
          "$ref": "../schemas.json#/schemas/MessageToBroadcast"
        }
      }
    }
  }
}
```

### Subscribe to messages stream

```json
{
  "asyncapi": "2.6.0",
  "info": {
    "title": "Messages stream API",
    "description": "Broadcasts received messages through an SSE connection to any subscribed user",
    "version": "1.0.0"
  },
  "servers": {
    "dev": {
      "url": "http://localhost:8080",
      "protocol": "http"
    }
  },
  "channels": {
    "/messages": {
      "description": "Subscribe to stream of messages",
      "subscribe": {
        "description": "Receive all incoming messages",
        "message": {
          "bindings": {
            "http": {
              "headers": {
                "type": "object",
                "additionalProperties": false,
                "required": [
                  "Content-Type", "X-SSE-Content-Type", "transfer-encoding"
                ],
                "properties": {
                  "Content-Type": {
                    "type": "string",
                    "enum": ["text/event-stream"]
                  },
                  "X-SSE-Content-Type": {
                    "type": "string",
                    "enum": ["application/json"]
                  },
                  "transfer-encoding": {
                    "type": "string",
                    "enum": ["chunked"]
                  }
                }
              }
            }
          },
          "$ref": "#/components/messages/Message"
        },
        "bindings": {
          "http": {
            "type": "request",
            "method": "GET"
          }
        }
      }
    }
  },
  "components": {
    "messages": {
      "Message": {
        "payload": {
          "$ref": "../schemas.json#/schemas/Message"
        }
      }
    }
  }
}
```

### Pub + Sub

```json
{
  "asyncapi": "2.6.0",
  "info": {
    "title": "Messages stream API",
    "description": "Broadcasts received messages through an SSE connection to any subscribed user",
    "version": "1.0.0"
  },
  "servers": {
    "dev": {
      "url": "http://localhost:8080",
      "protocol": "http"
    }
  },
  "channels": {
    "/messages": {
      "description": "Broadcast message",
      "publish": {
        "description": "Send message to broadcast to any subscribed user",
        "message": {
          "bindings": {
            "http": {
              "headers": {
                "type": "object",
                "additionalProperties": false,
                "required": [
                  "Content-Type"
                ],
                "properties": {
                  "Content-Type": {
                    "type": "string",
                    "enum": ["application/json"]
                  }
                }
              }
            }
          },
          "$ref": "#/components/messages/MessageToBroadcast"
        },
        "bindings": {
          "http": {
            "type": "request",
            "method": "POST"
          }
        }
      },
      "subscribe": {
        "description": "Receive all incoming messages",
        "message": {
          "bindings": {
            "http": {
              "headers": {
                "type": "object",
                "additionalProperties": false,
                "required": [
                  "Content-Type", "X-SSE-Content-Type", "transfer-encoding"
                ],
                "properties": {
                  "Content-Type": {
                    "type": "string",
                    "enum": ["text/event-stream"]
                  },
                  "X-SSE-Content-Type": {
                    "type": "string",
                    "enum": ["application/json"]
                  },
                  "transfer-encoding": {
                    "type": "string",
                    "enum": ["chunked"]
                  }
                }
              }
            }
          },
          "$ref": "#/components/messages/Message"
        },
        "bindings": {
          "http": {
            "type": "request",
            "method": "GET"
          }
        }
      }
    }
  },
  "components": {
    "messages": {
      "MessageToBroadcast": {
        "payload": {
          "$ref": "../schemas.json#/schemas/MessageToBroadcast"
        }
      },
      "Message": {
        "payload": {
          "$ref": "../schemas.json#/schemas/Message"
        }
      }
    }
  }
}
```

## Resume

Both specifications allow to you to describe Pub and Sub for SSE app, but with some tradeoffs

OpenAPI can't offer canonical way how to describe stream.

For example:
> ```json
> {
>   "type": "array",
>   "format": "event-stream",
>   "items": {
>     "$ref": "../schemas.json#/schemas/Message"
>   }
> }
> ```

What does it mean? Stream of messages? Stream of arrays with messages?

From other side AsyncAPI as expected can't offer flexible syntax for description of HTTP requests - headers, params location, status codes

It's up to you to choose which specification to use, but looks like it's better to use them both, instead of reinvent the wheel:
- [Swagger for Server Sent Events](https://github.com/OAI/OpenAPI-Specification/issues/396)
- [Swagger for WebSocket services](https://github.com/OAI/OpenAPI-Specification/issues/55#issuecomment-1057220490)

# References
- [Server-sent events](https://html.spec.whatwg.org/multipage/server-sent-events.html)
- [AsyncAPI](https://www.asyncapi.com/)
- [OpenAPI](https://www.openapis.org/)
