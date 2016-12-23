---
title: User Relations Basics
---

<a name="friends-and-followers"></a>
## User Relations (Friends & Followers)

### Relation directions

<!--- TODO: talks about directional and undirectional relation, and how friend and
follower are examples of them; discuss the values of SKYRelationDirection -->

Get all following users:

```obj-c
SKYQueryUsersOperation *operation = [SKYQueryUsersOperation queryUsersOperationByRelation:[SKYRelation followRelation] direction:SKYRelationDirectionActive];
operation.container = [SKYContainer defaultContainer];
[[SKYContainer defaultContainer] addOperation:operation];
```

Get all mutual followers: **[Not implemented]**

```obj-c
SKYQueryUsersOperation *operation = [SKYQueryUsersOperation queryUsersOperationByRelation:[SKYRelation followRelation] direction:SKYRelationDirectionMutual];
operation.container = [SKYContainer defaultContainer];
[[SKYContainer defaultContainer] addOperation:operation];
```

1. Two default relations: friend and follower

<a name="adding-relations"></a>
## Adding relation between users

```obj-c
SKYAddRelationsOperation *operation = [SKYAddRelationsOperation operationWithType:@"friend" usersToRelated:@[rick, ben]];
operation.container = [SKYContainer defaultContainer];
[[SKYContainer defaultContainer] addOperation:operation];
```

<a name="querying-relations"></a>
## Querying Relations

Get all friends:

```obj-c
SKYQueryUsersOperation *operation = [SKYQueryUsersOperation queryUsersOperationByRelation:[SKYRelation friendRelation]];
operation.container = [SKYContainer defaultContainer];
[[SKYContainer defaultContainer] addOperation:operation];
```

Get all followers:

```obj-c
SKYQueryUsersOperation *operation = [SKYQueryUsersOperation queryUsersOperationByRelation:[SKYRelation followRelation] direction:SKYRelationDirectionPassive];
operation.container = [SKYContainer defaultContainer];
__weak SKYQueryUsersOperation *weakOperation = operation;
operation.queryUserCompletionBlock = ^(NSArray *users, NSError *operationError) {
    NSLog(@"Operation will have overallCount after execution, %d", weakOperation.overallCount);
};

[[SKYContainer defaultContainer] addOperation:operation];
```

`SKYQueryUsersOperation-relationDirection` is only effective on `followRelation`.

<a name="removing-relations"></a>
## Removing Relations

```obj-c
SKYRemoveRelationsOperation *operation = [SKYRemoveRelationsOperation operationWithType:@"follower" usersToRemove:@[faseng, chima]];
operation.container = [SKYContainer defaultContainer];
[[SKYContainer defaultContainer] addOperation:operation];
```
