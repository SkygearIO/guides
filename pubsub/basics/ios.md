---
title: PubSub basics
---

[[toc]]

## Subscribing a channel

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[container.pubsub subscribeTo:@"hello" handler:^(NSDictionary *info) {
    NSString *name = info[@"name"];
    NSLog(@"%@ says hello", name);
}];
```

```swift
let container = SKYContainer.default()
container.pubsub.subscribe(to: "hello", handler: { (info) in
    let name = info?["name"]
    print("\(name) says hello")
})
```

## Publishing to a channel

```obj-c
[container.pubsub publishMessage:@{@"name": @"world"} toChannel:@"hello"];
```

```swift
container.pubsub.publishMessage(["name":"world"], toChannel: "hello")
```

To publish a message to the channel through cloud code, please refer to the
[Cloud Functions Guide: Calling Skygear API - Pubsub Events][doc-cloud-code-pubsub].

## Unsubscribing a channel

```obj-c
[container.pubsub unsubscribe:@"hello"];
```

```swift
container.pubsub.unsubscribe("hello")
```

Skygear will automatically re-connect on connection drop. Skygear will also
re-subscribe all existing handler on connection restore. So in normal case,
you don't need to re-subscribe all your handler on re-connect.

## Example: PING-PONG

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];

// Pinger
[container.pubsub subscribeTo:@"PING" handler:^(NSDictionary *info) {
    NSLog(@"Received a PING");
    [container.pubsub publishMessage:nil toChannel:@"PONG"];
}];

// Ponger
[container.pubsub subscribeTo:@"PONG" handler:^(NSDictionary *info) {
    NSLog(@"Received a PONG");
    [container.pubsub publishMessage:nil toChannel:@"PING"];
}];

// kick start the game
[container.pubsub publishMessage:nil toChannel:@"PING"];
```

```swift
let container = SKYContainer.default()

// Pinger
container.pubsub.subscribe(to: "PING", handler: { (info) in
    print ("Received a PING")
    container.pubsub.publishMessage(nil, toChannel: "PONG")
})

// Ponger
container.pubsub.subscribe(to: "PONG", handler: { (info) in
    print ("Received a PONG")
    container.pubsub.publishMessage(nil, toChannel: "PING")
})

// kick start the game
container.pubsub.publishMessage(nil, toChannel: "PING")
```

[doc-cloud-code-pubsub]: /guides/cloud-function/calling-skygear-api/python/#pubsub-events
