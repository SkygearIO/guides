---
title: PubSub basics
---

## Subscribing a channel

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[container.pubsubClient subscribeTo:@"hello" handler:^(NSDictionary *info) {
    NSString *name = info[@"name"];
    NSLog(@"%@ says hello", name);
}];
```

```swift
let container = SKYContainer.default()
container?.pubsubClient.subscribe(to: "hello", handler: { (info) in
    let name = info?["name"]
    print("\(name) says hello")
})
```

## Publishing to a channel

```obj-c
[container.pubsubClient publishMessage:@{@"name": @"world"} toChannel:@"hello"];
```

```swift
container?.pubsubClient.publishMessage(["name":"world"], toChannel: "hello")
```

To publish a message to the channel through cloud code, please refer to the
[Cloud Code Guide: PubSub Events][doc-cloud-code-pubsub].

## Unsubscribing a channel

```obj-c
[container.pubsubClient unsubscribe:@"hello"];
```

```swift
container?.pubsubClient.unsubscribe("hello")
```

Skygear will automatically re-connect on connection drop. Skygear will also
re-subscribe all existing handler on connection restore. So in normal case,
you don't need to re-subscribe all your handler on re-connect.

## Example: PING-PONG

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];

// Pinger
[container.pubsubClient subscribeTo:@"PING" handler:^(NSDictionary *info) {
    NSLog(@"Received a PING");
    [container.pubsubClient publishMessage:nil toChannel:@"PONG"];
}];

// Ponger
[container.pubsubClient subscribeTo:@"PONG" handler:^(NSDictionary *info) {
    NSLog(@"Received a PONG");
    [container.pubsubClient publishMessage:nil toChannel:@"PING"];
}];

// kick start the game
[container.pubsubClient publishMessage:nil toChannel:@"PING"];
```

```swift
let container = SKYContainer.default()
    
// Pinger
container?.pubsubClient.subscribe(to: "PING", handler: { (info) in
    print ("Received a PING")
    container?.pubsubClient.publishMessage(nil, toChannel: "PONG")
})
    
// Ponger
container?.pubsubClient.subscribe(to: "PONG", handler: { (info) in
    print ("Received a PONG")
    container?.pubsubClient.publishMessage(nil, toChannel: "PING")
})
    
// kick start the game
container?.pubsubClient.publishMessage(nil, toChannel: "PING")
```

[doc-cloud-code-pubsub]: /guides/cloud-code/calling-skygear-api/python#pubsub-events
