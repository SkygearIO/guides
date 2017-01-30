---
title: Offline Storage
---

<a name="cached-queries"></a>
## Cached Queries

Skygear provides a simple cached query mechanism for application that wants to display data when the device is offline, or when the app has just started and hasn't had enough time to complete query requests to Skygear service yet. To make use of this, you just need to provide an extra callback function to the `query` function, and the SDK will try to return the cached result before it gets the latest result from the Skygear server.

```obj-c
SKYQuery *query = [SKYQuery queryWithRecordType:@"todo" predicate:nil];
[[[SKYContainer defaultContainer] publicCloudDatabase] performCachedQuery:query completionHandler:^(NSArray *results, BOOL cached, NSError *error) {
    if (cached) {
        // results from cache
        // it will be called first if cache is available
    } else {
        // results from server, resulted in a second call
    }
}];
```

```swift
let query = SKYQuery(recordType: "todo", predicate: nil)
SKYContainer.default().publicCloudDatabase.performCachedQuery(query) { (results, cached, error) in
    if cached {
        // results from cache
        // it will be called first if cache is available
    } else {
        // results from server, resulted in a second call
    }
}
```