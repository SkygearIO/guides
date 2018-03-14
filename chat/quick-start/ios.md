---
title: Skygear Chat Quick Start
---

[[toc]]
Follow the steps below to add Skygear Chat to your app.

## Step 1: Add Skygear Core to your app

Skygear Core provides you the [cloud database](https://docs.skygear.io/guides/cloud-db/basics/js/) and the [user authentication](https://docs.skygear.io/guides/auth/basics/js/) modules. They are required modules for Skygear Chat.

Follow the [iOS quick start](/guides/intro/quickstart/ios/) guide to add Skygear Core to your app if you haven't.

## Step 2: Add Skygear Chat to your app using CocoaPods

1. Go to your `Podfile` and add the following lines to include `SKYKItChat` (Skygear Chat SDK) in your project.

```bash
# This line should exists after you follow the iOS Quick Start guide
pod 'SKYKit', '~> 1.4'

# Add this line
pod 'SKYKitChat', '~> 1.4'

# If you want to use UIKit too, include this line:
# pod 'SKYKitChat/UI', '~> 1.4'
```

Note: It is not a must to specify the version number (i.e. 1.1). If you don't, your app will be using the latest SDK version automatically. Check out all the version numbers [here](https://github.com/SkygearIO/skygear-SDK-iOS/releases).

2. Save your `Podfile` and run `pod install` in the terminal.

```bash
pod install
```
3. Then in `AppDelegate.m` or `AppDelegate.swift`, import the Skygear Chat SDK.

```obj-c
import <SKYKitChat/SKYKitChat.h>
```
```swift
import SKYKitChat
```

## Step 3: Enable the chat in your developer portal

Lastly, enable the chat module in your [developer portal](https://portal.skygear.io/apps). It is in the plug-ins page.

![Skygear plug-ins](/assets/common/enable-chat-plugin-on-portal.png)

Cool your are all set now.

## What's next from here

You got the basic. Next, learn more about:
* [Skygear Chat](https://docs.skygear.io/guides/chat/basics/ios/)
* [Skygear User Authentication](https://docs.skygear.io/guides/auth/basics/ios/)
* [Skygear Chat UIKit for iOS](https://github.com/SkygearIO/chat-SDK-iOS/blob/master/docs/UIKit/index.md) (common chat UI elements)
* [Chat demo projects for iOS](https://github.com/search?q=topic%3Askygear-chat+topic%3Askygear-ios+org%3Askygear-demo&type=Repositories)
