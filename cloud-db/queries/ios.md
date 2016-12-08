---
title: Queries
---

<a name="basic-queries"></a>
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

We constructed a `SKYQuery` to search for `todo` records. There are no additional
criteria needed so we put the predicate to `nil`. Then we assigned a
`NSSortDescription` to ask Skygear Server to sort the `todo` records by `order` field in ascending order.

<a name="conditions"></a>
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

If the key being queried is a JSON type, the `IN` operator can also be used to
query the key to check if it contains a particular value:

```obj-c
NSPredicate *inPredicate =
            [NSPredicate predicateWithFormat: @"%@ IN attribute", aValue];
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

### Full-text search (**NOT IMPLEMENTED**)

### References

`SKYKit` query supports filtering records by the reference field of records:

```obj-c
SKYReference *restaurantRef = [SKYReference referenceWithRecordID:[SKYRecordID recordIDWithRecordType:@"restaurant" name:@"my restaurant"]];
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"restaurant = %@", restaurantRef];
SKYQuery *query = [SKYQuery queryWithRecordType:@"order" predicate:predicate];

SKYDatabase *privateDB = database;

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

You can query by fields on a referenced record. Following the above example, if
we want to narrow orders placed in Italian restaurants only:

```obj-c
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"restaurant.cuisine = %@", @"italian"];
SKYQuery *query = [SKYQuery queryWithRecordType:@"order" predicate:predicate];

SKYDatabase *privateDB = database;

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

<a name="pagination-ordering"></a>
## Pagination and Ordering

### Sorting the records
We can sort records returned by:

```obj-c
NSSortDescriptor *sortDescriptor = [NSSortDescriptor sortDescriptorWithKey:@"_updated_at" ascending:NO];     // sorted by modificationDate
query.sortDescriptors = @[sortDescriptor];     // apply the NSSortDescriptor to the query
```

`SKYQuery` utilizes `NSPredicate` to apply filtering on query results. You can use other parameters to sort your queries.

### Limiting and Offset

We can limit the numbers of records returned by:

```obj-c
query.limit = 10;     // only show the top 10 records
```

We can also set an offset number to the query by:

```obj-c
query.offset = 5;     // ignore the first 5 records
```

Setting an `offset` number means skipping that many rolls before beginning to return rows. If the `offset` number is 0, then no rows will be skipped. If you use both `limit` and `offset`, then `offset` numbers of rows will be skipped before starting to limit the number of rows returned.

Now the first 5 records in the result list are skipped. The query result starts with the 6th record. It works just like SQL offset.

### Record Count

To get the number of all records matching a query, set the property
`overallCount` property of `SKYQuery` to `YES`. The record count can be
retrieved from `overallCount` property of `SKYQueryOperation` when
`perRecordCompletionBlock` is first called.

<a name="relational-queries"></a>
## Relational Queries

### Eager Loading

Skygear support eager loading of referenced records when you are querying the
referencing records. It's done by supplying a key path expression to
`[SKYQuery -transientIncludes]`:

```obj-c
SKYQuery *query = [SKYQuery queryWithRecordType:@"child" predicate:nil];
NSExpression *keyPath = [NSExpression expressionForKeyPath:@"parent"];
query.transientIncludes = @{@"parentRecord": keyPath};

[privateDB performQuery:query completionHandler:^(NSArray *results, NSError *error) {
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

It is possible to eager load records from multiple keys, but doing so will
impair performance.

### Reference Actions **[not implemented]**

`ON DELETE CASCADE` TBC.
