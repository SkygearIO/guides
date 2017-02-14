---
title: User Profile
---

<a name="user-profile"></a>
## User Profile

Whenever a new user signs up, a user profile is automatically created for
you to track user information other than their _username_, _email_ or _password_. _Username_, _email_ and _password_ are stored inside the reserved `_user` database, but user profile is stored in the public `user` database. They share the same column `_id` with exactly same value. You can access the
user profile the same way as accessing a record, and everything stored in
user profile is public and thus visible to any user.

```obj-c
SKYDatabase *publicDB = [[SKYContainer defaultContainer] publicCloudDatabase];
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"_id ==[c] %@", [container currentUserRecordID]];
SKYQuery *query = [SKYQuery queryWithRecordType:@"user" predicate:predicate];

[publicDB performQuery:query completionHandler:^(NSArray *results, NSError *error) {
    if (error) {
        NSLog(@"error quering user profile: %@", error);
        return;
    }

    NSLog(@"query successful");
    // do something else
}];
```

```swift
let publicDB = SKYContainer.default().publicCloudDatabase
let predicate = NSPredicate(format: "_id == [c] %@", SKYContainer.default().currentUserRecordID)
let query = SKYQuery(recordType: "user", predicate: predicate)
    
publicDB?.perform(query, completionHandler: { (results, error) in
    if error != nil {
        print ("error quering user profile: \(error)")
        return
    }
    
    print ("query successful")
    // do something else
})
```