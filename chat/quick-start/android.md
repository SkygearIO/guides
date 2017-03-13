---
title: Skygear Chat Quick Start
---

Follow the steps below to add Skygear Chat to your app.

## Step 1: Add Skygear Core to your app

Skygear Core provides you the [cloud database](https://docs.skygear.io/guides/cloud-db/basics/android/) and the [user authentication](https://docs.skygear.io/guides/auth/basics/android/) modules. They are required modules for Skygear Chat.

## Step 2: Add Skygear Chat to your app

You also need to install the Chat plugin SDK into your project via Gradle.

1. Add `io.skygear.plugins:chat:+` dependency in your `build.gradle` **of the module**.

```Java
dependencies {
    // other dependencies
    compile 'io.skygear.plugins:chat:+'
}
```

2. You will be hinted for a project sync as you have updated the `gradle` files. The Skygear Android SDK will have been installed when the sync is completed.

3. In each of your file to use Skygear Chat, make sure you have import the Skygear Chat SDK besides the core SDK.

```Java
import io.skygear.plugins.chat.*;
```

## Step 3: Enable the chat in your developer portal

Lastly, enable the chat module in your [developer portal](https://portal.skygear.io/apps). It is in the plug-ins page.

![Skygear plug-ins](/assets/common/enable-chat-plugin-on-portal.png)

Cool your are all set now.

## What's next from here

You got the basic. Next, learn more about:
* [Skygear Chat](https://docs.skygear.io/guides/chat/basics/android/)
* [Skygear User Authentication](https://docs.skygear.io/guides/auth/basics/android/)
