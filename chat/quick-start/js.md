---
title: Skygear Chat Quick Start
---

Follow the steps below to add Skygear Chat to your app.

## Step 1: Add Skygear Core to your app

Skygear Core provides you the [cloud database](https://docs.skygear.io/guides/cloud-db/basics/js/) and the [user authentication](https://docs.skygear.io/guides/auth/basics/js/) modules. They are required modules for Skygear Chat.

Follow the [JS quick start](https://docs.skygear.io/guides/quickstart/js/) guide to add Skygear Core to your app if you haven't.

## Step 2: Add Skygear Chat to your app

There are 2 ways to do it. Either use CDN or import the npm package.

### Method 1: CDN

Add the following lines into the header of your HTML file right under the CDNs of the Skygear Core.

```javascript
<script src ="http://code.skygear.io/js/chat/latest/skygear-chat.min.js" </script>
```

### Method 2: Npm Install

1. Add Skygear Chat to your project using npm install.
```bash
npm install skygear-chat
```
2. Then import `skygearchat` in your JS file.
```bash
import skygearchat from 'skygear-chat';
```

## Step 3: Enable the chat in your developer portal

Lastly, enable the chat module in your [developer portal](https://portal.skygear.io/apps). It is in the plug-ins page.

![Skygear plug-ins](/assets/common/enable-chat-plugin-on-portal.png)

Cool your are all set now.

## What's next from here

You got the basic. Next, learn more about:
* [Skygear Chat](https://docs.skygear.io/guides/chat/basics/js/) or [API reference](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/index.js~SkygearChatContainer.html)
* [Skygear User Authentication](https://docs.skygear.io/guides/auth/basics/js/)
