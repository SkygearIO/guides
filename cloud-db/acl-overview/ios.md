---
title: Access Control Overview
---

[[toc]]

Skygear supports access control on records. You can use it to control whether
a user can query, update or delete a record.

Skygear uses Access Control List (ACL) as the model of access control.
An ACL is a list of Access Control Entry (ACE), each ACE describes which
access rights a user target has to a record.

On top of ACL, each record has an owner, and **the owner has full
access** to the record.

There are 2 types of ACL in Skygear:

- Record-based ACL
- Field-based ACL

## Default ACL Settings

By default,

- read access is granted to the public, and
- write access is granted only to owner of the record.

```obj-c
// not logged in as any user
SKYQuery *query = [SKYQuery queryWithRecordType:@"note" predicate:nil];

SKYDatabase *publicDB = [[SKYContainer defaultContainer] publicCloudDatabase];
[publicDB performQuery:query completionHandler:^(NSArray *results, NSError *error) {
    if (error) {
        NSLog(@"error querying records: %@", error);
        return;
    }

    // Successfully query records
    SKYRecord *aNote = results[0];
    aNote[@"content"] = @"Hello Wolrd!";
    [publicDB saveRecord:aNote completion:^(SKYRecord * _Nullable record, NSError * _Nullable error) {
        if (error) {
            // You will get not Authenticated error here
            NSLog(@"error saving records: %@", error);
            return;
        }

        // You will NOT succeed
    }];
}];
```

```swift
// not logged in as any user
let query = SKYQuery(recordType: "note", predicate: nil)
let publicDB = SKYContainer.default().publicCloudDatabase

publicDB.perform(query) { (results, queryError) in
    if queryError != nil {
        print("Error: \(queryError?.localizedDescription)")
        return
    }

    // Successfully query records
    let aNote = notes[0] as SKYRecord
    aNote["content"] = "Hello World!"
    publicDB.save(aNote) { (results, saveError) in
        if saveError != nil {
            // You will get not Authenticated error here
            print("Error: \(saveError?.localizedDescription)")
            return
        }

        // You will NOT succeed
    }
}
```

## ACL Access Level

ACL in Skygear have 3 access level:

- No Access
- Read Only Access
- Read Write Access

### Read access

Read access allows querying and fetching records, which includes getting
all the fields of records as well as the access control settings.

### Write access

Write access allows saving and deleting records, which includes adding,
updating and removing all fields (**EXCEPT**
[reserved columns][doc-reserved-columns]) of records
as well as access control settings for the record.

## ACL User Target

### Public

Public means all other users and unauthenticated (not logged in) users.

### By User

Users are specified by User ID

### By Role

#### Define Roles

You can create different roles and later use them to design ACL targeting users
of specific roles.

```obj-c
SKYRole *manager = [SKYRole roleWithName:@"manager"];
SKYRole *visitor = [SKYRole roleWithName:@"visitor"];
```

```swift
let manager = SKYRole(name: "manager")
let visitor = SKYRole(name: "visitor")
```

You can set certain roles as admin so that they have the power to change other
users' roles.

```obj-c
SKYAuthContainer *auth = [[SKYContainer defaultContainer] auth];
[auth defineAdminRoles:@[manager]
            completion:^(NSError *error) {
              if (!error) {
                NSLog(@"Success!");
              } else {
                NSLog(@"Error: %@", error.localizedDescription);
              }
            }];
```

```swift
if let manager = manager as SKYRole! {
    SKYContainer.default().auth.defineAdminRoles([webmaster]) { (error) in
        if error != nil {
            print("Error: \(error?.localizedDescription)")
            return
        }

        print("Success!")
    }
}
```

You can also set the default roles so that newly signed up user would start
with these roles.

```obj-c
[auth setUserDefaultRole:@[visitor]
              completion:^(NSError *error) {
                if (!error) {
                  NSLog(@"Success!");
                } else {
                  NSLog(@"Error: %@", error.localizedDescription);
                }
              }];
```

```swift
if let visitor = visitor as SKYRole! {
    SKYContainer.default().auth.setUserDefaultRole([visitor]) { (error) in
        if error != nil {
            print("Error: \(error?.localizedDescription)")
            return
        }

        print("Success!")
    }
}
```

#### Assign and Revoke Roles

For admin user to change roles of other users:

```obj-c
// You must login as an admin user
NSArray<NSString *> *userIDs = @[ @"John", @"Ken" ];
NSArray<NSString *> *rolesToRevoke = @[ @"visitor" ];
NSArray<NSString *> *rolesToAssign = @[ @"manager" ];
SKYAuthContainer *auth = [[SKYContainer defaultContainer] auth];
[auth revokeRolesWithNames:rolesToRevoke
          fromUsersWihtIDs:userIDs
                completion:^(NSError * _Nullable revokeError) {
                    if (revokeError) {
                        NSLog(@"Error: %@", revokeError.localizedDescription);
                        return;
                    }

                    NSLog(@"Roles revoked");
                    [auth assignRoles:rolesToAssign
                              toUsers:userIDs
                           completion:^(NSError * _Nullable assignError) {
                               if (assignError) {
                                   NSLog(@"Error: %@", assignError.localizedDescription);
                                   return;
                               }

                               NSLog(@"Roles assigned");
                           }];
                }];
```

```swift
// You must login as an admin user
let userIDs: [String] = ["John", "Ken"]
let rolesToRevoke: [String] = ["visitor"]
let rolesToAssign: [String] = ["manager"]
let auth: SKYAuthContainer = SKYContainer.default().auth
auth.revokeRoles(withNames: rolesToRevoke, fromUsersWihtIDs: userIDs) { (revokeError) in
    if revokeError != nil {
        print("Error: \(revokeError?.localizedDescription)")
        return;
    }

    print("Roles revoked")
    auth.assignRoles(withNames: rolesToAssign, toUsersWithIDs: userIDs) { (assignError) in
        if assignError != nil {
            print("Error: \(assignError?.localizedDescription)")
            return;
        }

        print("Roles assigned")
    }
}
```

## Changing Default ACL Settings

You can change the default ACL settings:

```obj-c
SKYAccessControl *acl = [[SKYContainer defaultContainer] defaultAccessControl];
[acl setReadOnlyForPublic];
[acl setReadWriteAccessForRole:manager];
[[[SKYContainer defaultContainer] publicCloudDatabase] setDefaultAccessControl:acl];
```

```swift
let acl = SKYContainer.default().defaultAccessControl
acl?.setReadOnlyForPublic()
SKYContainer.default().publicCloudDatabase.defaultAccessControl = acl
```

After changing the default ACL setting, all records created in the future
will automatically have this ACL setting; however, ACL setting for existing
records created before this update will remain unchanged.

[doc-reserved-columns]: /guides/cloud-db/basics/js/#reserved-columns
