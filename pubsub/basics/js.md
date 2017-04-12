---
title: PubSub basics
---

[[toc]]

Pubsub will only work after the user has logged in. You can check the connection
by accessing `skygear.pubsub.connected`. Here, all the examples are using event
`ping`.

## Subscribing to an event

``` javascript
skygear.on('ping', (data) => {
  console.log(data);
});
```

### Waiting for a single event

``` javascript
skygear.pubsub.once('event')
.then(_ => {
  // do stuff after event happened
});
```
or with ES7 async/await:
``` javascript
await skygear.pubsub.once('event');
// do stuff after event happened
```

## Unsubscribing an event

### Unsubscribing all handlers for an event

``` javascript
skygear.off('ping');
```

### Unsubscribing a specific handler for an event

``` javascript
const handler = (data) => console.log(data);
skygear.on('ping', handler); // also returns handler

// later
skygear.off('ping', handler);
```

## Check if an event has handler

``` javascript
skygear.pubsub.hasHandlers('ping');
```

## Publishing an event

You can send a JavaScript object or native types (e.g. number, string, etc.) as
data, and all the skygear users (including yourself) will receive this data
with the correct type if they subscribed to event `ping`.

``` javascript
skygear.pubsub.publish('ping', data);
```

To publish a message to the channel through cloud code, please refer to the
[Cloud Functions Guide: Calling Skygear API - Pubsub Events][doc-cloud-code-pubsub].

## Listening to Connection state

Skygear will automatically re-connect on connection drop. Skygear will also 
re-subscribe all existing handler on connection restore. So in normal case,
you don't need to re-subscribe all your handler on re-connect.

We understand application may need to know the status of pubsub connection, we
provide `onOpen` and `onClose` callback for that.


``` javascript
function welcome() {
  console.log('Chat are ready!');
  // Don't subscribe channel here, or yield to multiple function call on same
  // event
}

function warning() {
  console.log('Chat is temporarily unavailable');
}

const welcomeHandle = skygear.pubsub.onOpen(welcome);
const alertHandle = skygear.pubsub.onClose(warning);

// You may cancel the registered callback by calling cancel
welcomeHandle.cancel();
alertHandle.cancel();
```

[doc-cloud-code-pubsub]: /guides/cloud-function/calling-skygear-api/python/#pubsub-events
