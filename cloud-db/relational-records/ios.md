---
title: Relational Records
---

[[toc]]

## Creating references

Skygear supports many-to-one (aka. parent-child) relation between records via _reference_.
`SKYReference` is a pointer to a record in database. Let's say we are going to
reference _Record A_ in _Record B_, we first construct a reference of Record A
using its id.

```obj-c
// aID is a placeholder of Record A's id
SKYReference *aRef = [SKYReference referenceWithRecordID:aID];
```

```swift
// aID is a placeholder of Record A's id
let aRef = SKYReference(recordID: aID)
```

Then assign this reference as a regular field of Record B:

```obj-c
// bRecord is a placeholder of Record B's object
bRecord[@"parent"] = aRef;
```

```swift
// bRecord is a placeholder of Record B's object
bRecord.setObject(aRef, forKey: "parent" as NSCopying!)
```


It will establish a reference from _Record B_ to _Record A_.


## Querying referenced records

This example shows how to query all notes (`Note` record) who has an `account` field reference to a user record. In this example, we will query all notes where `account` equals to the current user.

```obj-c
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

### Relational query by fields of reference record
You can query by fields on a referenced record. Following the above example, if
we want to query all notes where account's role is editor only:

```obj-c
NSPredicate *accountPredicate = [NSPredicate predicateWithFormat:@"account.role = %@", @"editor"];
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
let accountPredicate = NSPredicate(format: "account.role = %@", "editor")
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
})
```

### Relational query by record's ID
If you haven't have the corresponding record in hand (in this example, we will use the User record `182654c9-d205-43aa-8e74-d465c830087a`), you can reference with a specify `id` without making another query in this way:

```obj-c
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

## Eager loading

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
query.transientIncludes = ["parentRecord": keyPath]

SKYContainer.default().privateCloudDatabase.perform(query) { (results, error) in
    if error != nil {
        print ("error fetching child: \(error)")
        return
    }

    print ("received \(results?.count) children")
    for child in results as! [SKYRecord] {
        let parent: SKYRecord = child.transient.object(forKey: "parentRecord") as! SKYRecord
        print ("\(child.recordID)'s parent is \(parent.recordID)")
    }
}
```

It is possible to eager load records from multiple keys, but doing so will
impair performance.

## Deleting referenced records

`ON DELETE CASCADE` TBC.
