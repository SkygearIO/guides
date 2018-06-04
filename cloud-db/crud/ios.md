---
title: Basic CRUD
---

[[toc]]

Before working with the Skygear records, we suggest you reading the [basics][doc-cloud-db-basics].

## Creating records

### Creating a record

Let's imagine we are writing a To-Do app with Skygear. When user creates
an to-do item, we want to save that item on server. We probably will save that
to-do item like this:

```obj-c
SKYRecord *todo = [SKYRecord recordWithRecordType:@"todo"];
todo[@"title"] = @"Write documents for Skygear";
todo[@"order"] = @1;
todo[@"done"] = @NO;

SKYDatabase *privateDB = [[SKYContainer defaultContainer] privateCloudDatabase];
[privateDB saveRecord:todo completion:^(SKYRecord *record, NSError *error) {
    if (error) {
        NSLog(@"error saving todo: %@", error);
        return;
    }

    NSLog(@"saved todo with recordID = %@", record.recordID);
}];
```

```swift
let todo = SKYRecord(recordType: "todo")
todo.setObject("Write documents for Skygear", forKey: "title" as NSCopying!)
todo.setObject(1, forKey: "order" as NSCopying!)
todo.setObject(false, forKey: "done" as NSCopying!)

let privateDB = SKYContainer.default().privateCloudDatabase
privateDB.save(todo, completion: { (record, error) in
    if error != nil {
        print ("error saving todo: \(error)")
        return
    }

    print ("saved todo with record = \(record?.recordID)")
})

```

There are couples of things we have done here:

1. First we created a `todo` _record_ and assigned some attributes to it. you can use the `[]` subscript operator as shown above, or the `setObject:forKey:` method. Your app automatically creates this Class when you first use it.
2. We fetched the _container_ of our app, and took a reference to the private
   database of the current user. So when you save a _record_, you're saving the record to the private database of the current user.
3. We actually saved the `todo` record and registered a block to be executed
   after the action is done. When you have successfully saved a _record_, there are several fields automatically filled in for you, such as `SKYRecordID`, `recordName`,`creationDate` and `modificationDate`. A `SKYRecord` will be returned and you can make use of the block to add additional logic which will run after the save completes.

### Creating multiple records

You can also save multiple `SKYRecord`s at once with
[`saveRecords`][api-save-records] or
[`saveRecordsAtomically`][api-save-records-atomically] which make sure the save
operation either succeeds or fails as a whole:

```obj-c
SKYRecord *noteOne = [SKYRecord recordWithRecordType:@"note"];
SKYRecord *noteTwo = [SKYRecord recordWithRecordType:@"note"];

NSArray *notesToSave = [[NSArray alloc] initWithObjects: noteOne, noteTwo, nil];

SKYDatabase *privateDB = [[SKYContainer defaultContainer] privateCloudDatabase];
[privateDB saveRecords:(notesToSave) completionHandler:^(NSArray *savedRecords, NSError *operationError) {
    if (operationError) {
        // Error completing the operation
        NSLog(@"error completing operation");
        return;
    }

    NSLog(@"saved all the todo records");
} perRecordErrorHandler:^(SKYRecord *record, NSError *error) {
    if (error) {
        // Error saving an individual record
        NSLog(@"error saving todo: %@", error);
        return;
    }

    NSLog(@"saved todo with recordID = %@", record.recordID);
}];
```

```swift
let noteOne = SKYRecord(recordType: "note")
let noteTwo = SKYRecord(recordType: "note")

let notesToSave = [noteOne, noteTwo]

let privateDB = SKYContainer.default().privateCloudDatabase
privateDB.saveRecords(notesToSave, completionHandler: { (savedRecords, operationError) in
    if operationError != nil {
        // Error completing the operation
        print ("error completing operation")
        return
    }

    print ("saved all the todo records")
}, perRecordErrorHandler: { (record, error) in
    if error != nil {
        // Error saving an individual record
        print ("error saving todo: \(error)")
        return
    }

    print ("saved todo with recordID = \(record?.recordID)")
})
```
## Updating records

Now let's return to our to-do item example. This is how you save a `SKYRecord`:

```obj-c
SKYRecord *todo = [SKYRecord recordWithRecordType:@"todo"];
todo[@"title"] = @"Write documents for Skygear";
todo[@"order"] = @1;
todo[@"done"] = @NO;

SKYDatabase *privateDB = [[SKYContainer defaultContainer] privateCloudDatabase];
[privateDB saveRecord:todo completion:^(SKYRecord *record, NSError *error) {
    if (error) {
        NSLog(@"error saving todo: %@", error);
        return;
    }

    NSLog(@"saved todo with recordID = %@", record.recordID);
}];
```

```swift
let todo = SKYRecord(recordType: "todo")
todo.setObject("Write documents for Skygear", forKey: "title" as NSCopying!)
todo.setObject(1, forKey: "order" as NSCopying!)
todo.setObject(false, forKey: "done" as NSCopying!)

let privateDB = SKYContainer.default().privateCloudDatabase
privateDB.save(todo, completion: { (record, error) in
    if error != nil {
        print ("error saving todo: \(error)")
        return
    }

    print ("saved todo with record = \(record?.recordID)")
})
```

After you have successfully saved the `SKYRecord`, the server will return an updated `SKYRecord`. Your console should look like this:

```
2015-09-22 16:16:37.893 todoapp[89631:1349388] saved todo with recordID = <SKYRecordID: 0x7ff93ac37940; recordType = todo, recordName = 369067DC-BDBC-49D5-A6A2-D83061D83BFC>
```

As you can see, the returned `SKYRecord` now has a `recordID`. The `recordID` property on your saved todo `SKYRecord` is a unique `recordID` which identifies the record in a database.

With the `recordID` you can modify the record later on. Say if
you have to mark this todo as done:

```obj-c
SKYRecord *todo = [SKYRecord recordWithRecordType:@"todo" name:@"369067DC-BDBC-49D5-A6A2-D83061D83BFC"];
todo[@"done"] = @YES;
[[[SKYContainer defaultContainer] privateCloudDatabase] saveRecord:todo completion:nil];
```

```swift
let todo = SKYRecord(recordType: "todo", name: "CCD9C879-E45D-4377-8D86-562A04E4D2CD")
todo.setObject(true, forKey: "done" as NSCopying!)
SKYContainer.default().privateCloudDatabase.save(todo, completion: nil)
```

Note that the data in the returned record in the completion block may be
different from the originally saved record. This is because additional
fields maybe applied on the server side when the record is saved (e.g. the updated `modificationDate`). You may want to inspect the returned record for any changes applied on the server side.


## Querying records

With the `recordID` we could also fetch the record from a database:

```obj-c
SKYRecordID *recordID = [SKYRecordID recordIDWithRecordType:@"todo" name:@"369067DC-BDBC-49D5-A6A2-D83061D83BFC"];
[privateDB fetchRecordWithID:recordID completionHandler:^(SKYRecord *record, NSError *error) {
    if (error) {
        NSLog(@"error fetching todo: %@", error);
        return;
    }

    NSString *title = record[@"title"];
    NSNumber *order = record[@"order"];
    NSNumber *done = record[@"done"];

    NSLog(@"Fetched a note (title = %@, order = %@, done = %@)", title, order, done);
}];
```

```swift
let recordID = SKYRecordID(recordType: "todo", name: "369067DC-BDBC-49D5-A6A2-D83061D83BFCD")
SKYContainer.default().privateCloudDatabase.fetchRecord(with: recordID) { (record, error) in
    if error != nil {
        print ("error fetching todo: \(error)")
        return
    }

    let title = record?.object(forKey: "title")
    let order = record?.object(forKey: "order")
    let done = record?.object(forKey: "done")

    print ("Fetched a note (title = \(title), order = \(order), done = \(done)")

}
```

In Objective-C, to get the values out of the `SKYRecord`, you can use the `[]` subscript operator as shown above, or the `objectForKey:` method:

```obj-c
NSString *title = [record objectForKey: @"title"];
NSNumber *order = [record objectForKey: @"order"];
NSNumber *done = [record objectForKey: @"done"];
```

You can construct a `SKYQuery` object by providing a `recordType`. You can configure the `SKYQuery` by mutating its state. Read the [More About Queries][doc-queries] section to learn more.

## Deleting records

### Deleting a record

Deleting a record requires its `recordID` too:

```obj-c
SKYRecordID *recordID = [SKYRecordID recordIDWithRecordType:@"todo" name:@"369067DC-BDBC-49D5-A6A2-D83061D83BFC"];
[[[SKYContainer defaultContainer] privateCloudDatabase] deleteRecordWithID:recordID completionHandler:nil];
```

```swift
let recordID = SKYRecordID(recordType: "todo", name: "369067DC-BDBC-49D5-A6A2-D83061D83BFC")
SKYContainer.default().privateCloudDatabase.deleteRecord(with: recordID, completionHandler: nil)
```

If you are to delete records in batch, you could also use the
`SKYDatabase-deleteRecordsWithIDs:completionHandler:perRecordErrorHandler:`
method.

### Deleting multiple records

You can also delete multiple records at once:

```obj-c
SKYRecordID *noteOneRecordID = [SKYRecordID recordIDWithRecordType:@"todo" name:@"369067DC-BDBC-49D5-A6A2-D83061D83BFC"];
SKYRecordID *noteTwoRecordID = [SKYRecordID recordIDWithRecordType:@"todo" name:@"348275VF-SKGF-69DK-10FH-D83061D83BFC"];

NSArray *notesToDeleteRecordID = [[NSArray alloc] initWithObjects: noteOneRecordID, noteTwoRecordID, nil];

[[[SKYContainer defaultContainer] privateCloudDatabase] deleteRecordsWithIDs: (notesToDeleteRecordID) completionHandler:^(NSArray *deletedRecordIDs, NSError *error) {
    if (error) {
        // Error completing the operation
        NSLog(@"error completing operation");
        return;
    }

    NSLog(@"deleted all the todo records");
} perRecordErrorHandler:^(SKYRecordID *recordID, NSError *error) {
    if (error) {
        // Error deleting an individual record
        NSLog(@"error deleting todo: %@", error);
        return;
    }

    NSLog(@"deleting todo with recordID = %@", recordID);
}];
```

```swift
let noteOneRecordID = SKYRecordID(recordType: "todo", name: "369067DC-BDBC-49D5-A6A2-D83061D83BFC")
let noteTwoRecordID = SKYRecordID(recordType: "todo", name: "348275VF-SKGF-69DK-10FH-D83061D83BFC")

let notesToDeleteRecordID = [noteOneRecordID, noteTwoRecordID]

SKYContainer.default().privateCloudDatabase.deleteRecords(withIDs: notesToDeleteRecordID, completionHandler: { (deletedRecordIDs, error) in
    if error != nil {
        // Error completing the operation
        print ("error completing operation")
        return
    }
    print ("deleted all the todo records")
}) { (recordID, error) in
    if error != nil {
        print ("error deleting todo: \(error)")
        return
    }

    print ("deleting todo with recordID = \(recordID)")
}
```


[doc-cloud-db-basics]:/guides/cloud-db/basics/ios/
[doc-queries]: /guides/cloud-db/queries/ios/
[api-save-records]: https://docs.skygear.io/ios/reference/latest/Classes/SKYDatabase.html#/c:objc(cs)SKYDatabase(im)saveRecords:completionHandler:perRecordErrorHandler:
[api-save-records-atomically]: https://docs.skygear.io/ios/reference/latest/Classes/SKYDatabase.html#/c:objc(cs)SKYDatabase(im)saveRecordsAtomically:completionHandler:
