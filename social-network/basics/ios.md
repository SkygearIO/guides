---
title: User Relations Basics
---

[[toc]]


## User Relations (Friends & Followers)

### Relation directions

<!--- TODO: talks about directional and undirectional relation, and how friend and
follower are examples of them; discuss the values of SKYRelationDirection -->

Get all following users:

```obj-c
SKYQueryOperation *operation = [SKYQueryOperation queryUsersOperationByRelation:[SKYRelation followingRelation]];
operation.container = [SKYContainer defaultContainer];
[[SKYContainer defaultContainer] addOperation:operation];
```

```swift
let operation = SKYQueryOperation.queryUsersOperation(by: SKYRelation.following())
operation.container = SKYContainer.default()
SKYContainer.default().add(operation)
```

Get all mutual followers: **[Not implemented]**

```obj-c
SKYQueryOperation *operation = [SKYQueryOperation queryUsersOperationByRelation:[SKYRelation friendRelation]];
operation.container = [SKYContainer defaultContainer];
[[SKYContainer defaultContainer] addOperation:operation];
```

```swift
let operation = SKYQueryOperation.queryUsersOperation(by: SKYRelation.friend())
operation.container = SKYContainer.default()
SKYContainer.default().add(operation)
```

1. Two default relations: friend and follower


## Adding relation between users

```obj-c
SKYAddRelationsOperation *operation = [SKYAddRelationsOperation operationWithType:@"friend" usersToRelated:@[rick, ben]];
operation.container = [SKYContainer defaultContainer];
[[SKYContainer defaultContainer] addOperation:operation];
```

```swift
let operation = SKYAddRelationsOperation(type: "friend", usersToRelated: [rick!, ben!])
operation.container = SKYContainer.default()
SKYContainer.default().add(operation)
```


## Querying Relations

Get all friends:

```obj-c
SKYQueryOperation *operation = [SKYQueryOperation queryUsersOperationByRelation:[SKYRelation friendRelation]];
operation.container = [SKYContainer defaultContainer];
[[SKYContainer defaultContainer] addOperation:operation];
```

```swift
let operation = SKYQueryOperation.queryUsersOperation(by: SKYRelation.friend())
operation.container = SKYContainer.default()
SKYContainer.default().add(operation)
```

Get all followers:

```obj-c
SKYQueryOperation *operation = [SKYQueryOperation queryUsersOperationByRelation:[SKYRelation followedRelation]];
operation.container = [SKYContainer defaultContainer];
__weak SKYQueryOperation *weakOperation = operation;
operation.queryRecordsCompletionBlock = ^(NSArray *users, SKYQueryCursor *queryCursor, NSError *error) {
    NSLog(@"Operation will have overallCount after execution, %d", weakOperation.overallCount);
};
    
[[SKYContainer defaultContainer] addOperation:operation];
```

```swift
let operation = SKYQueryOperation.queryUsersOperation(by: SKYRelation.followed())
operation.container = SKYContainer.default()
weak var weakOperation = operation
operation.queryRecordsCompletionBlock = { (users, queryCursor, error) in
    print ("Operation will have overall count after execution, \(weakOperation.overallCount)")
}

SKYContainer.default().add(operation)
```

`SKYQueryUsersOperation-relationDirection` is only effective on `followRelation`.


## Removing Relations

```obj-c
SKYRemoveRelationsOperation *operation = [SKYRemoveRelationsOperation operationWithType:@"follower" usersToRemove:@[faseng, chima]];
operation.container = [SKYContainer defaultContainer];
[[SKYContainer defaultContainer] addOperation:operation];
```

```swift
let operation = SKYRemoveRelationsOperation(type: "follower", usersToRemove: [faseng!, chima!])
operation.container = SKYContainer.default()
SKYContainer.default().add(operation)
```
