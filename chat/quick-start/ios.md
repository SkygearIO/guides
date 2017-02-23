---
title: Skygear Chat Quick Start
---
Follow the steps below to add Skygear Chat to your app.

## Step 1: Add Skygear Core to your app

Skygear Core provides you the [cloud database](https://docs.skygear.io/guides/cloud-db/basics/js/) and the [user authentication](https://docs.skygear.io/guides/auth/basics/js/) modules. They are required modules for Skygear Chat.

Follow the [iOS quick start](https://docs.skygear.io/guides/quickstart/iOS/) guide to add Skygear Core to your app if you haven't.

## Step 2: Add Skygear Chat to your app using CocoaPods

1. Go to your Podfile and add the following lines to include SKYKItChat (Skygear Chat SDK) in your project.

```bash
pod 'SKYKitChat', :git => 'https://github.com/SkygearIO/chat-SDK-iOS.git'
```
2. Save your Podfile and run `pod install` in the terminal.

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
