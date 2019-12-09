# Skygear SDK for web

### Installation

Skygear SDK for web is available on npm package registry:

```bash
$ npm install @skygear/web
```

### Configuration

To connect the SDK to your app, configure the SDK at app launch:

```typescript
import skygear from "@skygear/web";

async function configureSkygear() {
    // TODO: Use endpoint and API key for your app.
    await skygear.configure({
        endpoint: "<App Endpoint>",
        apiKey: "<API Key>",
    });
}
```



