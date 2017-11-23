---
title: Skygear Push Basics
---

[[toc]]

You need to register the devices before sending them push notifications.

<a id="registering-device"></a>
## Registering device

It is suggested that you register a device with remote server on every launch.
Since the container remembers the device ID when the device is registered
on the remote server, it will reuse an existing device ID whenever you try
to register the current device.

You should also request a remote notification token at some point in the
program. If it is appropriate to ask the user for permission for remote
notification, you can also do so when the application launches.

The method to register the device varies depends on the framework you use
with the JavaScript SDK.

When the device is registered on the remote server, a device ID will be
available to the client application. If you register the current device
using the above convenient methods, the device ID is returned from
`deviceID` property on the container.

### React-Native on iOS

You should [add PushNotificationIOS to your project][ios-client] which helps
you by requesting permission to obtain push notification token.

When the device obtained the push notification token, the `register`
event is fired. Add an event listener for this event and use the
`registerDevice` function to register the token with Skygear.

```js
var { PushNotificationIOS } = require('react-native');
var skygear = require('skygear');

PushNotificationIOS.addEventListener('register', function(token) {
  skygear.push.registerDevice(arg);
});
```

### React-Native on Android

You should follow [Set up a GCM Client App on Android][gcm-client] and use
[GCM for React Native Android][react-native-gcm-android] to obtain the push
notification token.

When the device obtained the push notification token, the `register`
event is fired. Add an event listener for this event and use the
`registerDevice` function to register the token with Skygear.

```js
var GcmAndroid = require('react-native-gcm-android');
var skygear = require('skygear');

GcmAndroid.addEventListener('register', function(token) {
  skygear.push.registerDevice(arg);
});
```


## Sending push notification from cloud code

To send push notifications through cloud code, please refer to the
[Cloud Code Guide: Push Notifications][doc-cloud-function-push-notifications].



## Sending push notification to users

```js
var skygear = require('skygear');

skygear.push.sendToUser(
  ['2aa4af2a-699a-4e43-8d67-7598757fc7ed'], // User IDs
  {
    'apns': {
        'aps': {
            'alert': {
                'title': title,
                'body': message,
            }
        },
        'from': 'skygear',
        'operation': 'notification',
    },
    'gcm': {
         'notification': {
              'title': title,
              'body': message,
          }
    },
  }
);
```


## Sending push notification to devices

```js
var skygear = require('skygear');

skygear.push.sendToDevice(
  ['2aa4af2a-699a-4e43-8d67-7598757fc7ed'], // Device IDs
  {
    'apns': {
        'aps': {
            'alert': {
                'title': title,
                'body': message,
            }
        },
        'from': 'skygear',
        'operation': 'notification',
    },
    'gcm': {
         'notification': {
              'title': title,
              'body': message,
          }
    },
  }
);
```


## Unregistering device

When a device is registered, it is associated with the authenticated user.
Therefore it is necessary to unregister the device when the user is no longer
associated with the device, for example when the user logs out.

```js
var skygear = require('skygear');

skygear.unregisterDevice();
```


[doc-cloud-function-push-notifications]: /guides/cloud-function/calling-skygear-api/python/#push-notifications
[ios-client]: https://facebook.github.io/react-native/docs/pushnotificationios.html
[gcm-client]: https://developers.google.com/cloud-messaging/android/client
[react-native-gcm-android]: https://github.com/oney/react-native-gcm-android
