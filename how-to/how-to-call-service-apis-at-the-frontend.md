# How to call API of a deployed service on Skygear

## Introduction

The APIs of your microservices on Skygear can be called with the `fetch` method provided in Skygear JS SDK, or with tools like cURL that can make HTTP-based requests.

## Prerequisites

Before you start this section, please ensure you have `skycli` installed and configured properly. If not, follow [this](../set-up/set-up-steps.md) to set it up.

You will need your service\(s\) deployed on Skygear before you can call them, where `skycli` is the only way to perform deployments.

**TL**;**DR** Sooner or later you will need to install and configure `skycli`.

## Call APIs using the Skygear JS SDK

We recommend you to use `fetch` in Skygear JS SDK to call your APIs, since it not only manages the users sessions but also throws in required headers for you.

Assume you already have a web frontend and a up-and-running microservice on Skygear.

### **Install the SDK in the frontend client**

The CDN way:

```text
<script src="https://unpkg.com/@skygear/web"></script>
```

The `npm install` or `yarn install` way:

```bash
$ npm install @skygear/web
# OR
$ yarn add @skygear/web
```

### **Configure the SDK with your app's endpoint and API key**

```javascript
skygear.configure({
  endpoint: '<INSERT API ENDPOINT HERE>',
  apiKey: '<INSERT API KEY HERE>'
});
```

### **Fetch!**

Assuming that a microservice is deployed under the path `api` and would like to send a request to one of its APIs `fetch_blogs`, we can achieve this with Skygear SDK as shown below:

```javascript
skygear
    .fetch('api/fetch_blogs', { method: 'POST' })
    .then((resp) => resp.json())
    .then((posts) => console.log(posts));
```

This API `fetch_blogs` actually exists in one of our Skygear demos. Visit this [guide](../quick-started/integrate-with-frontend.md) for more information.

## Call APIs using cURL

Similarly, you can curl the exact same API:

```text
curl -X POST \
  https://<your-app>.v2.dev.skygearapis.com/api/fetch_blogs \
  -H 'Cache-Control: no-cache' \
  -H 'Connection: keep-alive' \
  -H 'Content-Length: 33' \
  -H 'accept: */*' \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -H 'sec-fetch-mode: cors' \
  -H 'sec-fetch-site: same-site' \
  -H 'x-skygear-api-key: <your_app_key>' \
```

The request is visibly bulkier, plus extra effort is spent to substitute values like a Skygear API key. This is nevertheless the go-to option when you would like to test out an API quickly without having to install some SDKs and initialise a new project.

