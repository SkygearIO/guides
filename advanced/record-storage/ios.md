---
title: Local Storage (Offline)
---

[[toc]]

## Setup

Record storage relies on [Query][doc-queries]

Add `SKYContainerDelegate` to your controller.

```obj-c
- (void)container:(SKYContainer *)container didReceiveNotification:(SKYNotification *)notification
{
    // ...
    [[SKYRecordStorageCoordinator defaultCoordinator] handleUpdateWithRemoteNotification:notification];
}

```
```swift
func container(_ container: SKYContainer!, didReceive notification: SKYNotification!)
{
	// ...
	SKYRecordStorageCoordinator.default().handleUpdate(withRemoteNotification: notification)
}
```

## Creating a record storage

```obj-c
SKYQuery *query = [[SKYQuery alloc] initWithRecordType:@"note" predicate:nil];
SKYRecordStorageCoordinator *coordinator = [SKYRecordStorageCoordinator defaultCoordinator];
SKYRecordStorage* recordStorage = [coordinator recordStorageWithDatabase:self.database
                                                                  query:query options:nil];
```

```swift
let query = SKYQuery(recordType: "note", predicate: nil)
let coordinator = SKYRecordStorageCoordinator.default()
let recordStorage = coordinator.recordStorage(with: SKYContainer.default().privateCloudDatabase,
											  query: query, options: nil)
```

## Saving records

```obj-c
SKYRecord *note = [SKYRecord recordWithRecordType:@"note"];
note[@"content"] = @"record storage is fun!";
[recordStorage saveRecord:note];
```

```swift
let note = SKYRecord(recordType: "note")
note.setValue("record storage is fun!", forKey: "content" as NSCopying!)
recordStorage.save(note)
```


## Deleting records

```obj-c
[recordStorage deleteRecord:noteToDelete];
```

```swift
recordStorage.delete(note)
```

## Fetching records

```obj-c
SKYRecord *record = [recordStorage recordWithRecordID:recordID];
```

```swift
let record = recordStorage.record(with: recordID)
```

## Querying records

### Without Predicate

```obj-c
for (SKYRecord *note in [recordStorage recordsWithType:@"note"]) {
    // do something with note
}
```
```swift
let notes = recordStorage.records(withType: "note")
for note in notes as! [SKYRecord]{
    // do something with note
}
```

### With Predicate

```obj-c
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"done == false"];
NSArray *records = [recordStorage recordsWithType:@"todo"
                                        predicate:predicate
                                  sortDescriptors:nil];
for (SKYRecord *note in records) {
    // do something with note
}
```
```swift
let predicate = NSPredicate(format: "done == false")
let records = recordStorage.records(withType: "todo", predicate: predicate, sortDescriptors: nil)
for note in records as! [SKYRecord] {
    // do something with note
}
```

## Listening to change event

```obj-c
[[NSNotificationCenter defaultCenter] addObserverForName:SKYRecordStorageDidUpdateNotification
                                                  object:recordStorage
                                                   queue:[NSOperationQueue mainQueue]
                                              usingBlock:^(NSNotification *note) {
                                                  self.notes = [self.categoryStorage recordsWithType:@"note"];
                                                  [self.tableView reloadData];
                                              }];
```

```swift
NotificationCenter.default.addObserver(
	forName: Notification.Name.SKYRecordStorageDidUpdate,
	object: recordStorage,
	queue: OperationQueue.main) { (note) in
   		notes = categoryStorage.records(withType: "note")
    	self.tableView.reloadData()
}
```
