---
description: >-
  Installable packages that eases the communication between your apps and
  Skygear.
---

# Install SkygearSDK

## Installation

Currently there are three Skygear SDKs available in JavaScript/TypeScript:

* **@skygear/web**: all-purpose web frontend SDK
* **@skygear/node-client**: Package that allows you to easily integrate Skygear with servers written in Node.js 
* **@skygear/react-native**: specifically made for React Native apps 

Below is a command demonstrating how one can be installed:

```text
$ npm install @skygear/web
```

## Configuration

To import and use a SDK in your app, configure it at the launch stage:

```javascript
import skygear from "@skygear/web";

async function configureSkygear() {
    await skygear.configure({
        endpoint: "<your_app_endpoint>",
        apiKey: "<your_api_key>",
    });
}
```

The Skygear container is imported as `skyear` in the code sample, where is then configured with a user-specific endpoint and api key. If the container hasn't been configured properly, the SDK functions won't work.

## Interact with AuthGear

One main functionality of the Skygear SDKs is to communicate with AuthGear in an intuitive manner,  just like:

```javascript
import skygear from "@skygear/web";

// please configure the container properly first
    
skygear.auth.login("some_registered_username", "some_registered_password")
    .then(status -> {
        // your code...
    })
```

All user operations such as sign-up, log-in and log-out with AuthGear can be performed through the SDK.

## Interact with Deployed Apps - Fetch API

The [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) is an interface allowing users to retrieve web resources with ease. We have shipped this with Skygear SDKs where your app endpoint is automatically taken care for you. 

Here an example to visualize this:

```javascript
import skygear from "@skygear/web";

// please configure the container properly first

skygear.apiClient
  .fetch("/some-api", {
    method: "POST",
    headers: {
      "x-skygear-api-key": "<your_api_key>"
    }
  })
  .then(result => {
    console.log(result);
  });
```

Say you have an API at `https://<your_app_name>.skygearapp.com/some-api` and some of your code need to retrieve things from it. Within the Skygear container \(already configured, of course\) is an apiClient with has a method called `fetch`. This is essentially the one from Fetch API, just that you only have to provide the api subpath `/some-api` instead of the full URL.

## Customization

While the SDK methods aim to supply a more user-friendly and simpler interface for Skygear users to integrate their apps, they and the classes can indeed be extended to customize their behaviours. To find out the details of them, go to the [documentation](https://skygeario.github.io/skygear-SDK-JS/docs/index.html).

## Docs

To know more about the SDKs, please refer to the [documentation](https://skygeario.github.io/skygear-SDK-JS/docs/index.html).



