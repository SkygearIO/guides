---
title: User Record
---

[[toc]]


## Introduction

Whenever a new user signs up, a user record is automatically created for
you to track user information other than their username, email or other
user data.

The user record is created using the record type `user` with
the column `_id` storing the user ID, so you can use it
in the same way as using any other record types.
You can query and update a user record by manipulating using
the `user` record type.

Important note: This user record is created in the public database, i.e.
it is visible to any other user. Therefore you should not store any sensitive
information in this record type. You will need to create another record type
with the private database for information that can only be accessed by the
owner.

## Saving to the current user record

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
SKYRecord *record = [SKYRecord recordWithRecordType:@"user"
                                               name:container.auth.currentUserRecordID];
record[@"language"] = @"en-US";
record[@"gender"] = @"male";
record[@"age"] = @20;

[container.publicCloudDatabase saveRecord:record
                               completion:^(SKYRecord *record, NSError *error) {
                                   if (error != nil) {
                                       NSLog(@"Error: %@", error);
                                       return;
                                   }
                                   NSLog(@"%@", record); // updated user record
                               }];
}];
```

```swift
let container = SKYContainer.default()
let record = SKYRecord(recordType: "user", name: container.auth.currentUserRecordID)
record.setObject("en-US", forKey: "language" as NSCopying)
record.setObject("male", forKey: "gender" as NSCopying)
record.setObject(20, forKey: "age" as NSCopying)
container.publicCloudDatabase.save(record!) { (record, error) in
    if error != nil {
        print(error!)
    }
    print(record!) // updated user record
}
```

## Retrieving the current user record

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[container.publicCloudDatabase fetchRecordWithID:[SKYRecordID recordIDWithRecordType:@"user", name:container.auth.currentUserRecordID]
                               completionHandler:^(SKYRecord *record, NSError *error) {
                                   if (error != nil) {
                                       NSLog(@"Error: %@", error);
                                       return;
                                   }
                                   NSLog(@"%@", record); // user record
                               }];
}];
```

```swift
let container = SKYContainer.default()
container.publicCloudDatabase.fetchRecord(SKYRecordID.init(recordType: "user", name: container.auth.currentUserRecordID)) { (record, error) in
    if error != nil {
        print(error!)
    }
    print(record!) // user record
}
```

<a id="search-users"></a>

## Retrieving another user record by email or username

You can retrieve the public records of other users by using their emails or
usernames. You can provide either a single email/username or an array of
emails/usernames.

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"email = %@", @"ben@oursky.com"];
SKYQuery *query = [SKYQuery queryWithRecordType:@"user" predicate:predicate];
[container.publicCloudDatabase performQuery:query completionHandler:^(NSArray *records, NSError *error) {
    if (error) {
        NSLog(@"error querying user: %@", error);
        return;
    }

    for (SKYRecord *user in users) {
       NSLog(@"%@", user); // user record
    }
}];
```

```swift
let predicate = NSPredicate(format: "email = %@", "ben@oursky.com"!)
let query = SKYQuery(recordType: "user", predicate: predicate)
SKYContainer.container.publicCloudDatabase.perform(query, completionHandler: { (users, error) in
    if error != nil {
        print ("error querying user: \(error)")
        return
    }

    for user in users as! [SKYRecord] {
       print(user); // user record
    }
})
```

To find user by username:

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"username = %@", @"ben"];
SKYQuery *query = [SKYQuery queryWithRecordType:@"user" predicate:predicate];
[container.publicCloudDatabase performQuery:query completionHandler:^(NSArray *records, NSError *error) {
    if (error) {
        NSLog(@"error querying user: %@", error);
        return;
    }

    for (SKYRecord *user in users) {
       NSLog(@"%@", user); // user record
    }
}];
```

```swift
let predicate = NSPredicate(format: "username = %@", "ben"!)
let query = SKYQuery(recordType: "user", predicate: predicate)
SKYContainer.container.publicCloudDatabase.perform(query, completionHandler: { (users, error) in
    if error != nil {
        print ("error querying user: \(error)")
        return
    }

    for user in users as! [SKYRecord] {
       print(user); // user record
    }
})
```
