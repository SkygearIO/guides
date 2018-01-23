---
title: Access Control Overview
---

[[toc]]

Skygear has provided a default security setting on all records.

By default,

- read access is granted to the public, and
- write access is granted only to owner of the record.

Read access includes access to **query** and **fetch**. Write access includes access to **update** and **delete**. More details below.

For example:

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

If you want to customize these security settings, Skygear supports access control on records and fields level. You can use it to control whether a user can query, update or delete a record or a field in the record.

Skygear uses [Access Control List][wiki-acl] (ACL) as the model of access control.
An ACL is a list of Access Control Entry (ACE), each ACE describes which
access rights a user target has to a record.

Each ACE consists of:
- **Resource**, which defines what is affected by the entry.
- **Access level**, which defines what actions can be performed (for example, read or write).
- **User Target**, which defines who can perform the specified actions (for example, a specific user or group of users).


These are the concepts you need to know:
- Record ownership, **the owner has full access** to the record.
- The 2 types of ACL in Skygear: [Record-based ACL][doc-record-acl] and [Field-based ACL][doc-field-acl].

## How Field-based ACL works with Record-based ACL

Default Field-based ACL is read and write access for public. So if you use Record-based ACL 
**WITHOUT** Field-based ACL, you will be able to read or save all fields of the record as long 
as you have read or write access in Record-based ACL respectively.

### Fetch and Query

For each fetch or query operation, Record-based ACL will affect **if you can, or cannot,
successfully get the record**, while Field-based ACL will affect **what you can see** in the
record.

For e.g. you set a record with public read write, and all the record's
field are no access to everyone. You will still get a list of records, but
you can only read the reserved fields of the records.

### Save

In atomic save, when a single record or a single field is failed to save,
the whole operation will be rolled back.

In non-atomic save, when a single record or a single field is failed to save,
that part would be skipped, and the rest of the operation will still succeed.

## ACL User Target

Both [Record-based ACL][doc-record-acl] and [Field-based ACL][doc-field-acl] allow you to set
specific permission toward different user target. Below are the types of User Target you can
define.

- **Public**: refers to all other users of the app, unauthenticated (not logged in) users included
- **By User**: users specified by User ID
- **By Role**: users grouped into specific roles by admin

Below we will talk about how to create roles and assign a role to users.

## Role in Skygear

Roles can only be defined, assigned or revoked by user in **admin** role. You can create the first
admin user of your application by calling Cloud Functions with master key. Another way to define
your first admin users are by the User Management in the CMS of Skygear.io

### Define role

You can create different roles and later use them to design ACL targeting users of specific roles.

```obj-c
SKYRole *manager = [SKYRole roleWithName:@"manager"];
SKYRole *visitor = [SKYRole roleWithName:@"visitor"];
```

```swift
let manager = SKYRole(name: "manager")
let visitor = SKYRole(name: "visitor")
```

### Admin role

In Skygear roles are either admin role or non-admin role. By default, there is an admin role called `Admin`.

What only admin role can do:

- Assign role to user
- Revoke role to user

:::tips
For security concern, only admin can change the role of a user, thus the user must request another user with admin role to change the role for him.

If a group is just used for display purpose, like group member. Then the group is a public group. Users actually do not need permission from Admin to join the group.

Do NOT overload user roles to model group if no access control is necessary. It will have performance penalty since it involves an extra check during query/save.
:::


### Setting admin role and default role in development

:::caution
Admin role and default role must be set during development. The two operations are banned in production environment.
:::

You define new roles to be admin role.

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

You can also set the default roles so that newly signed up user would start with these roles.

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

### Assign and Revoke Roles

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
                    [auth assignRolesWithNames:rolesToAssign
                                toUsersWithIDs:userIDs
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

## Next Part
- [Record-based ACL][doc-record-acl]
- [Field-based ACL][doc-field-acl]

[wiki-acl]: https://en.wikipedia.org/wiki/Access_control_list
[doc-reserved-columns]: /guides/cloud-db/basics/ios/#reserved-columns
[doc-record-acl]: /guides/cloud-db/record-acl/ios/
[doc-field-acl]: /guides/cloud-db/field-acl/
[doc-field-acl-matching]: /guides/cloud-db/field-acl/#resource-matching
