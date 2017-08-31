---
title: Queries
---

[[toc]]


## Basic Queries

We have shown how to fetch individual records by ids, but in real-world
application there are usually needs to show a list of items according to
some criteria. It is supported by queries in Skygear.

Let's see how to fetch a list of to-do items to be displayed in our
hypothetical To-Do app:

```obj-c
SKYQuery *query = [SKYQuery queryWithRecordType:@"todo" predicate:nil];

NSSortDescriptor *sortDescriptor = [NSSortDescriptor sortDescriptorWithKey:@"order" ascending:YES];
query.sortDescriptors = @[sortDescriptor];

SKYDatabase *privateDB = [[SKYContainer defaultContainer] privateCloudDatabase];
[privateDB performQuery:query completionHandler:^(NSArray *results, NSError *error) {
    if (error) {
        NSLog(@"error querying todos: %@", error);
        return;
    }

    NSLog(@"Received %@ todos.", @(results.count));
    for (SKYRecord *todo in results) {
        NSLog(@"Got a todo: %@", todo[@"title"]);
    }
}];
```

```swift
let query = SKYQuery(recordType: "todo", predicate: nil)
let sortDescriptor = NSSortDescriptor(key: "order", ascending: true)
query?.sortDescriptors = [sortDescriptor]

SKYContainer.default().privateCloudDatabase.perform(query) { (results, error) in
    if error != nil {
        print ("error querying todos: \(error)")
        return
    }

    print ("Received \(results?.count) todos.")
    for todo in results as! [SKYRecord] {
        print ("Got a todo \(todo["title"])")
    }
}
```

We constructed a `SKYQuery` to search for `todo` records. There are no additional
criteria needed so we put the predicate to `nil`. Then we assigned a
`NSSortDescription` to ask Skygear Server to sort the `todo` records by `order` field in ascending order.


## Conditions

To use `SKYQuery` with ease, we recommend using the methods provided to add constraints. However, you can also use `NSPredicate` to add constraints if you wish. The following features are supported:

### Basic Comparisons

### Strings

### IN

The `IN` operator can be used to query a key for value that matches one of the
item in an `NSArray`.

```obj-c
NSPredicate *inPredicate =
            [NSPredicate predicateWithFormat: @"attribute IN %@", aCollection];
```

```swift
let inPredicate = NSPredicate(format: "attribute IN %@", aCollection)
```

If the key being queried is a JSON type, the `IN` operator can also be used to
query the key to check if it contains a particular value:

```obj-c
NSPredicate *inPredicate =
            [NSPredicate predicateWithFormat: @"%@ IN attribute", aValue];
```

```swift
let inPredicate = NSPredicate(format: "%@ IN attribute", aValue)
```

### Relation Predicate

The `SKYRelationPredicate` can be used to query for records having a relation with
the current user. For this kind of query, the record have an relation with
the current user if the record has an attribute that contains a user having
the relation with the current user.

For example, to query for records owned by a user that the current user is following:

```obj-c
NSPredicate *p =
            [SKYRelationPredicate predicateWithRelation:[SKYRelation followingRelation]
                                                    key:@"_owner"]
```

```swift
let p = SKYRelationPredicate(relation: SKYRelation.following(), keyPath: "_owner")
```

### Relational Queries

This example shows how to query all notes (`Note` record) who has an `account` field reference to a user record. In this example, we will query all notes where `account` equals to the current user.

```objective-c
// You should have logged in
SKYRecord *currentUser = [SKYContainer defaultContainer].auth.currentUser;
SKYReference *nameRef = [SKYReference referenceWithRecord:currentUser];

NSPredicate *accountPredicate = [NSPredicate predicateWithFormat:@"account = %@", nameRef];

SKYQuery *query = [SKYQuery queryWithRecordType:@"Note" predicate: accountPredicate];


SKYDatabase *publicDB = [[SKYContainer defaultContainer] publicCloudDatabase];
[publicDB performQuery:query completionHandler:^(NSArray *results, NSError *error) {
    if (error) {
        NSLog(@"error querying notes: %@", error);
        return;
    }

    NSLog(@"Received %@ notes.", @(results.count));
    for (SKYRecord *note in results) {
        NSLog(@"Got a note: %@", note[@"title"]);
    }
}];

```

```swift
// You should have logged in
let currentUser = SKYContainer.default().auth.currentUser
let nameRef = SKYReference(recordID: (currentUser?.recordID)!)
let accountPredicate = NSPredicate(format: "account = %@", nameRef)

let query = SKYQuery(recordType: "Note", predicate: accountPredicate)

SKYContainer.default().publicCloudDatabase.perform(query) { (results, error) in
    if error != nil {
        print ("error querying note: \(error)")
        return
    }
    
    print ("Received \(results?.count) notes.")
    
    for note in results as! [SKYRecord] {
        print ("Got a Note  \(note["content"])")
    }
}
```

If you haven't have the corresponding record in hand (in this example, we will use the User record `182654c9-d205-43aa-8e74-d465c830087a`), you can reference with a specify `id` without making another query in this way:

```objective-c
SKYReference *nameRef = [SKYReference referenceWithRecordID:[SKYRecordID recordIDWithCanonicalString:@"account/182654c9-d205-43aa-8e74-d465c830087a"]];

NSPredicate *accountPredicate = [NSPredicate predicateWithFormat:@"account = %@", nameRef];

SKYQuery *query = [SKYQuery queryWithRecordType:@"Note" predicate: accountPredicate];


SKYDatabase *publicDB = [[SKYContainer defaultContainer] publicCloudDatabase];
[publicDB performQuery:query completionHandler:^(NSArray *results, NSError *error) {
    if (error) {
        NSLog(@"error querying notes: %@", error);
        return;
    }

    NSLog(@"Received %@ notes.", @(results.count));
    for (SKYRecord *note in results) {
        NSLog(@"Got a note: %@", note[@"title"]);
    }
}];

```

```swift

let nameRef = SKYReference(recordID: SKYRecordID(canonicalString: "account/182654c9-d205-43aa-8e74-d465c830087a"))
let accountPredicate = NSPredicate(format: "account = %@", nameRef)

let query = SKYQuery(recordType: "Note", predicate: accountPredicate)

SKYContainer.default().publicCloudDatabase.perform(query) { (results, error) in
    if error != nil {
        print ("error querying notes: \(error)")
        return
    }
    
    for note in results as! [SKYRecord] {
        print ("Got a Note  \(note["content"])")
    }
}

```


### Full-text search (**NOT IMPLEMENTED**)

`SKYKit` query supports filtering records by the reference field of records:

```obj-c
SKYReference *restaurantRef = [SKYReference referenceWithRecordID:[SKYRecordID recordIDWithRecordType:@"restaurant" name:@"my restaurant"]];
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"restaurant = %@", restaurantRef];
SKYQuery *query = [SKYQuery queryWithRecordType:@"order" predicate:predicate];

SKYDatabase *privateDB =  [[SKYContainer defaultContainer] privateCloudDatabase];

[privateDB performQuery:query completionHandler:^(NSArray *orders, NSError *error) {
    if (error) {
        NSLog(@"error querying orders: %@", error);
        return;
    }

    for (SKYRecord *order in orders) {
        // work with the fetched order
    }
}];
```
```swift
let restaurantRef = SKYReference(recordID: SKYRecordID(recordType: "restaurant", name: "my restaurant"))
let predicate = NSPredicate(format: "restaurant = %@", restaurantRef!)
let query = SKYQuery(recordType: "order", predicate: predicate)

let privateDB = SKYContainer.default().privateCloudDatabase

privateDB.perform(query, completionHandler: { (orders, error) in
    if error != nil {
        print ("error querying orders: \(error)")
        return
    }

    for order in orders as! [SKYRecord] {
        // work with the fetched order
    }
})
```

You can query by fields on a referenced record. Following the above example, if
we want to narrow orders placed in Italian restaurants only:

```obj-c
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"restaurant.cuisine = %@", @"italian"];
SKYQuery *query = [SKYQuery queryWithRecordType:@"order" predicate:predicate];

SKYDatabase *privateDB =  [[SKYContainer defaultContainer] privateCloudDatabase];

[privateDB performQuery:query completionHandler:^(NSArray *orders, NSError *error) {
    if (error) {
        NSLog(@"error querying orders: %@", error);
        return;
    }

    for (SKYRecord *order in orders) {
        // work with the fetched order
    }
}];
```

```swift
let predicate = NSPredicate(format: "restaurant.cuisine = %@", "italian")
let query = SKYQuery(recordType: "order", predicate: predicate)

let privateDB = SKYContainer.default().privateCloudDatabase

privateDB.perform(query, completionHandler: { (orders, error) in
    if error != nil {
        print ("error querying orders: \(error)")
        return
    }

    for order in orders as! [SKYRecord] {
        // work with the fetched order
    }
})
```


## Pagination and Ordering

### Sorting the records
We can sort records returned by:

```obj-c
SKYQuery *query = [SKYQuery queryWithRecordType:@"order" predicate:nil];
NSSortDescriptor *sortDescriptor = [NSSortDescriptor sortDescriptorWithKey:@"_updated_at" ascending:NO];     // sorted by modificationDate
query.sortDescriptors = @[sortDescriptor];     // apply the NSSortDescriptor to the query
```

```swift
let query = SKYQuery(recordType: "order", predicate: nil)
let sortDescriptor = NSSortDescriptor(key: "_updated_at", ascending: false)     // sorted by modificationDate
query?.sortDescriptors = [sortDescriptor]     // apply the NSSortDescriptor to the query
```

`SKYQuery` utilizes `NSPredicate` to apply filtering on query results. You can use other parameters to sort your queries.

### Limiting and Offset

We can limit the numbers of records returned by:

```obj-c
query.limit = 10;     // only show the top 10 records
```

```swift
query?.limit = 10	  // only show the top 10 records
```

We can also set an offset number to the query by:

```obj-c
query.offset = 5;     // ignore the first 5 records
```

```swift
query?.offset = 5	  // ignore the first 5 records
```

Setting an `offset` number means skipping that many rolls before beginning to return rows. If the `offset` number is 0, then no rows will be skipped. If you use both `limit` and `offset`, then `offset` numbers of rows will be skipped before starting to limit the number of rows returned.

Now the first 5 records in the result list are skipped. The query result starts with the 6th record. It works just like SQL offset.

### Record Count

To get the number of all records matching a query, set the property
`overallCount` property of `SKYQuery` to `YES`. The record count can be
retrieved from `overallCount` property of `SKYQueryOperation` when
`perRecordCompletionBlock` is first called.


## Relational Queries

### Eager Loading

Skygear support eager loading of referenced records when you are querying the
referencing records. It's done by supplying a key path expression to
`[SKYQuery -transientIncludes]`:

```obj-c
SKYQuery *query = [SKYQuery queryWithRecordType:@"child" predicate:nil];
NSExpression *keyPath = [NSExpression expressionForKeyPath:@"parent"];
query.transientIncludes = @{@"parentRecord": keyPath};

[[[SKYContainer defaultContainer] privateCloudDatabase] performQuery:query completionHandler:^(NSArray *results, NSError *error) {
    if (error) {
        NSLog(@"error fetching child: %@", error);
        return;
    }

    NSLog(@"received %@ children", @(results.count));
    for (SKYRecord *child in results) {
        SKYRecord *parent = child.transient[@"parentRecord"];
        NSLog(@"%@'s parent is %@", child.recordID, parent.recordID);
    }
}];
```

```swift
let query = SKYQuery(recordType: "child", predicate: nil)
let keyPath = NSExpression(forKeyPath: "parent")
query?.transientIncludes = ["parentRecord": keyPath]

SKYContainer.default().privateCloudDatabase.perform(query) { (results, error) in
    if error != nil {
        print ("error fetching child: \(error)")
        return
    }

    print ("received \(results?.count) childern")
    for child in results as! [SKYRecord] {
        let parent: SKYRecord = child.transient.object(forKey: "parentRecord") as! SKYRecord
        print ("\(child.recordID)'s parent is \(parent.recordID)")
    }
}
```

It is possible to eager load records from multiple keys, but doing so will
impair performance.

### Reference Actions **[not implemented]**

`ON DELETE CASCADE` TBC.
