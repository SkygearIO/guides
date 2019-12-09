---
description: >-
  There are two ways you can call service APIs at the frontend: using the
  Skygear JS SDK or plain HTTP request. Learn how they work.
---

# How to call service APIs at the frontend

## Call functions using the Skygear JS SDK

We recommend you to use the Skygear JS SDK to call functions at the frontend since it manages the user sessions for you. 

Assume you already have a web frontend and a up-and-running Microservice on Skygear.

**1. Install the SDK in the frontend client.** 

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

**2. Configure the Skygear SDK** with the app endpoint and the API key of the app when your app initialises.

```javascript
skygear.configure({
  endpoint: '<INSERT API ENDPOINT HERE>',
  apiKey: '<INSERT API KEY HERE>'
});
```

**3. Call the service APIs using the `fetch` function**. Assume we already have a function named `fetch_blogs` deployed to Skygear and we want to call it at the frontend.

```javascript
skygear
    .fetch('api/fetch_blogs', { method: 'POST' })
    .then((resp) => resp.json())
    .then((posts) => console.log(posts));
```

## Call service APIs using HTTP requests

=== require updates ===

