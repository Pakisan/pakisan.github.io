---
title: Axios with OpenTelemetry
description: How to trace requests with Axios and OpenTelemetry
date: 2024-08-25 15:06:00 +0400
author: pakisan
categories: [OpenTelemetry, Axios]
tags: [openapi, http, axios]
image:
  path: /assets/assets/img/2024-08-25-axios-with-opentelemetry/cover image.png
---

Right now I'm consulting folks, who are building new EdTech platform.

As a big fan of API specifications - [OpenAPI](https://www.openapis.org), [AsyncAPI](https://www.asyncapi.com), I decided to start developing from contracts

## API client initialization

For OpenAPI I choose [openapi-client-axios](https://openapistack.co/docs/openapi-client-axios/intro/). Few lines of code
and I have pre-built Axios client for interaction with our platform API

Let's define out client

```js
const chatAPI = new OpenAPIClientAxios({
  definition: '/chat-api.json',
  axiosConfigDefaults: {},
});
```

Now it's time to initialize it

```js
onMounted(async () => {
  chatAPIClient.value = await chatAPI.init();
  chatService.value = new ChatService(chatAPI, chatAPIClient.value);

  chats.value = await chatService.value.requestChats();
})
```

And, it's done. Now we have typed client, generated from OpenAPI specification

## OpenTelemetry initialization

When API client is ready, it's type to wrap it up with OpenTelemetry, to start sending traces to our APM.

I'll use Open-Source version of [SigNoz](https://signoz.io) APM

### Let's define our OpenTelemetry

Installing OpenTelemetry dependencies

```shell
nmp i @opentelemetry/sdk-trace-web \
  @opentelemetry/context-zone \
  @opentelemetry/resources \
  @opentelemetry/semantic-conventions \
  @opentelemetry/exporter-trace-otlp-http \
  @opentelemetry/core \
  @opentelemetry/api
```

### Configuring resource

After installing of OpenTelemetry dependencies, it's time to configure our resource

```ts
import {Resource} from '@opentelemetry/resources';
import {
    SEMRESATTRS_DEPLOYMENT_ENVIRONMENT,
    SEMRESATTRS_SERVICE_NAME,
    SEMRESATTRS_SERVICE_VERSION,
} from '@opentelemetry/semantic-conventions';

const resource = Resource.default().merge(
  new Resource({
    [SEMRESATTRS_SERVICE_NAME]: '{application_name}', // for example: chat
    [SEMRESATTRS_SERVICE_VERSION]: '{application_version}', // for example: 23.78.0-RC2
    [SEMRESATTRS_DEPLOYMENT_ENVIRONMENT]: '{application_environment}', // for example: prod, dev
  })
);
```

### Configuring provider

When resource is ready, it's time to configure provider

```ts
import {BatchSpanProcessor, WebTracerProvider} from '@opentelemetry/sdk-trace-web';
import {OTLPTraceExporter} from "@opentelemetry/exporter-trace-otlp-http";
import {ZoneContextManager} from '@opentelemetry/context-zone';
import {W3CTraceContextPropagator} from "@opentelemetry/core";

const provider = new WebTracerProvider({ resource });
// Uncomment to see traces in web console
// provider.addSpanProcessor(new SimpleSpanProcessor(new ConsoleSpanExporter()));
provider.addSpanProcessor(
  new BatchSpanProcessor(
    new OTLPTraceExporter({
      url: 'http://{signoz_address}:4318/v1/traces',
    })
  )
);
provider.register({
  // Changing default contextManager to use ZoneContextManager - supports asynchronous operations - optional
  contextManager: new ZoneContextManager(),
  propagator: new W3CTraceContextPropagator(), // Send in W3C format
});
```

### Configuring tracing

Because Axios doesn't use [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API), we can't automatically send traces via extensions.

That's why we need to introduce proxy function, which will send traces to our APM

```ts
import {Operation} from "openapi-client-axios";
import {AxiosResponse} from "axios";

export const webTracerWithZone = provider.getTracer('{application_name}');

export async function traceSpan<F extends (...args: any) => Promise<AxiosResponse<any, any>>>(
    operation: Operation,
    operationBaseUrl: string,
    apiCall: F
): Promise<AxiosResponse<any, any>> {
    return webTracerWithZone.startActiveSpan(operation.operationId!, async (span: Span) => {
        try {
            console.info(`requesting ${operation.operationId}`);
            const apiResponse: AxiosResponse<any, any> = await apiCall();

            span.setAttribute('http.method', operation.method);
            span.setAttribute('http.url', `${operationBaseUrl}${operation.path}`);
            span.setAttribute('http.status_code', apiResponse.status);

            return apiResponse;
        } catch (error) {
            span.setStatus({code: SpanStatusCode.ERROR, message: `${error}`});
            throw error;
        } finally {
            span.end();
        }
    })
}
```

### Resulted OpenTelemetry configuration

Here is full OpenTelemetry configuration

```ts
import {BatchSpanProcessor, WebTracerProvider} from '@opentelemetry/sdk-trace-web';
import {ZoneContextManager} from '@opentelemetry/context-zone';
import {Resource} from '@opentelemetry/resources';
import {
    SEMRESATTRS_DEPLOYMENT_ENVIRONMENT,
    SEMRESATTRS_SERVICE_NAME,
    SEMRESATTRS_SERVICE_VERSION,
} from '@opentelemetry/semantic-conventions';
import {OTLPTraceExporter} from "@opentelemetry/exporter-trace-otlp-http";
import {W3CTraceContextPropagator} from "@opentelemetry/core";
import {Span, SpanStatusCode} from "@opentelemetry/api";
import {Operation} from "openapi-client-axios";
import {AxiosResponse} from "axios";

const resource = Resource.default().merge(
    new Resource({
        [SEMRESATTRS_SERVICE_NAME]: '{application_name}', // for example: chat
        [SEMRESATTRS_SERVICE_VERSION]: '{application_version}', // for example: 23.78.0-RC2
        [SEMRESATTRS_DEPLOYMENT_ENVIRONMENT]: '{application_environment}', // for example: prod, dev
    })
);

const provider = new WebTracerProvider({ resource });
// provider.addSpanProcessor(new SimpleSpanProcessor(new ConsoleSpanExporter()));
provider.addSpanProcessor(
    new BatchSpanProcessor(
        new OTLPTraceExporter({
            url: 'http://{signoz_address}:4318/v1/traces',
        })
    )
);
provider.register({
    // Changing default contextManager to use ZoneContextManager - supports asynchronous operations - optional
    contextManager: new ZoneContextManager(),
    propagator: new W3CTraceContextPropagator(),
});

export const webTracerWithZone = provider.getTracer('{application_name}');

export async function traceSpan<F extends (...args: any) => Promise<AxiosResponse<any, any>>>(
    operation: Operation,
    operationBaseUrl: string,
    apiCall: F
): Promise<AxiosResponse<any, any>> {
    return webTracerWithZone.startActiveSpan(operation.operationId!, async (span: Span) => {
        try {
            console.info(`requesting ${operation.operationId}`);
            const apiResponse: AxiosResponse<any, any> = await apiCall();

            span.setAttribute('http.method', operation.method);
            span.setAttribute('http.url', `${operationBaseUrl}${operation.path}`);
            span.setAttribute('http.status_code', apiResponse.status);

            return apiResponse;
        } catch (error) {
            span.setStatus({code: SpanStatusCode.ERROR, message: `${error}`});
            throw error;
        } finally {
            span.end();
        }
    })
}
```

## Tracing API client requests

Let's assume that we have `ChatService` to wrap common logic

To trace requests we need to add next dependencies

```ts
import {traceSpan} from "@/opentelemetry.ts";

import {Client, Components} from "@api/chatapi";
import {AxiosResponse} from "axios";
import {OpenAPIClientAxios, Operation} from "openapi-client-axios";
import {ExplicitParamValue, HttpMethod} from "openapi-client-axios/types/client";
```

and add proxy function, like this

```ts
private async apiCall(
    operationId: string,
    parameters: Array<ExplicitParamValue> | undefined = undefined,
    data: {} | undefined = undefined
): Promise<AxiosResponse<any, any>> {
    const operation: Operation = this.chatApi.getOperation(operationId)!
    const operationBaseUrl: string = this.chatApi.getBaseURL(operation)!;

    return await traceSpan(
        operation,
        operationBaseUrl,
        async (): Promise<AxiosResponse<any, any>> => {
            if (operation.method === HttpMethod.Get) {
                // @ts-ignore No index signature with a parameter of type string was found on type PathsDictionary
                return await this.chatAPIClient.paths[`${operation.path}`].get(parameters);
            } else {
                // @ts-ignore No index signature with a parameter of type string was found on type PathsDictionary
                return await this.chatAPIClient.paths[`${operation.path}`].post(parameters, data);
            }
        });
    }
```

now we can make request like this

```ts
public async requestChats(): Promise<Chat[]> {
    try {
        const apiResponse = await this.apiCall('getChats'); // operationId from OpenAPI
        return this.handleRequestChats(apiResponse);
    } catch (error) {
        console.error(error);
        return [];
    }
}
```

here is result

<p align="center">
  <img src="/assets/assets/img/2024-08-25-axios-with-opentelemetry/trace example.png" alt="trace example">
</p>
