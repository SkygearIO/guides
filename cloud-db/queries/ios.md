---
title: More About Queries
---

[[toc]]


## Basic queries

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
query.sortDescriptors = [sortDescriptor]

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


## Query conditions

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

### Social Relation Predicate

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

## Pagination and ordering

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
query.sortDescriptors = [sortDescriptor]     // apply the NSSortDescriptor to the query
```

`SKYQuery` utilizes `NSPredicate` to apply filtering on query results. You can use other parameters to sort your queries.

### Limiting and Offset

We can limit the numbers of records returned by:

```obj-c
query.limit = 10;     // only show the top 10 records
```

```swift
query.limit = 10	  // only show the top 10 records
```

We can also set an offset number to the query by:

```obj-c
query.offset = 5;     // ignore the first 5 records
```

```swift
query.offset = 5	  // ignore the first 5 records
```

Setting an `offset` number means skipping that many rolls before beginning to return rows. If the `offset` number is 0, then no rows will be skipped. If you use both `limit` and `offset`, then `offset` numbers of rows will be skipped before starting to limit the number of rows returned.

Now the first 5 records in the result list are skipped. The query result starts with the 6th record. It works just like SQL offset.

## Record counts

To get the number of all records matching a query, set the property
`overallCount` property of `SKYQuery` to `YES`. The record count can be
retrieved from `overallCount` property of `SKYQueryOperation` when
`perRecordCompletionBlock` is first called.

## Getting the reserved columns

For each record type stored in the database, a table with the same name as the record type is created. For example, if your record type is called `note`, there is a table called `note` in the database. Each row in the table corresponds to one record.

For each record table there exists two types of columns, those that are reserved by Skygear and those that are user-defined. Reserved columns contain metadata of a record, such as record ID, record owner and creation time. Names of reserved columns are prefixed with underscore (`_`).

It is possible to manipulate data in record tables directly. However, one should exercise cautions when modifying data directly in record tables.

Each record table contains the following reserved columns:

| Column Name   | Object Attribute           | Description                                     |
|---------------|----------------------------|-------------------------------------------------|
| `_created_at` | `creationDate`             | `NSDate` object of when record was created      |
| `_updated_at` | `modificationDate`         | `NSDate` object of when record was updated      |
| `_created_by` | `creatorUserRecordID`      | `NSString` object of user id of record creator  |
| `_updated_by` | `lastModifiedUserRecordID` | `NSString` object of user id of record updater  |
| `_owner`      | `ownerUserRecordID`        | `NSString` object of user id of owner           |
| `_id`         | `recordID`                 | `SKYRecordID` object of record id               |

You can retrieve the values from the object by accessing its properties:

```obj-c
NSDate *creationDate = [noteObject creationDate];
NSString *creatorID = [noteObject creatorUserRecordID];
SKYRecordID *recordID = [record recordID];
NSString *recordType = [record recordType];
```

```swift
let creationDate = noteObject.creationDate
let creatorID = noteObject.creatorUserRecordID
let record = record.recordID
let recordType = record.recordType
```

Please head to [Database Schema][doc-database-schema] to read more about Reserved Columns, Record Tables and Reserved Tables.

[doc-database-schema]:/guides/advanced/database-schema/
