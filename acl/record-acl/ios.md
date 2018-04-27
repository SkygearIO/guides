---
title: Record-based ACL
---

[[toc]]

Record-based ACL is useful when users have different access to each record.

Take Google doc as an example, you can only access a doc (a record) when the right is granted to you by the owner.

In Skygear, each record has a `_access` field, which stores the ACL of that record. You may update the ACL of a record by calling ACL API on the record and save it afterward.

Scroll to the bottom you will find an example of the application of the Record-based ACL.

Now let's have a look at how to set Record-based ACL.

:::tips
To understand record-based ACL you need basic Skygear ACL concepts. Learn about it [here][doc-overview-acl] if you haven't.
:::

## Record-based ACL of Public

Public ACL (including `NoAccessForPublic`, `ReadOnlyForPublic` and
`ReadWriteAccessForPublic`) define the permission unauthenticated users have. You can change the record
ACL for Public users as follows:

```obj-c
SKYDatabase *publicDB = [[SKYContainer defaultContainer] publicCloudDatabase];

SKYRecord *secretNote = [SKYRecord recordWithRecordType:@"note"];
secretNote[@"content"] = @"I am your father";
[secretNote.accessControl setNoAccessForPublic];
[publicDB filed:secretNote completion:/* ... */];

SKYRecord *publicNote = [SKYRecord recordWithRecordType:@"note"];
publicNote[@"content"] = @"Hello world!";
[publicNote.accessControl setReadWriteAccessForPublic];
[publicDB saveRecord:publicNote completion:/* ... */];
```
```swift
let publicDB = SKYContainer.default().publicCloudDatabase

let secretNote = SKYRecord(recordType: "note")
secretNote.setObject("I am your father", forKey: "content" as NSCopying!)
secretNote.accessControl?.setNoAccessForPublic()
publicDB.save(secretNotefile{ /* ... */ }

let publicNote = SKYRecord(recordType: "note")
publicNote.setObject("Hello World!", forKey: "content" as NSCopying!)
publicNote.accessControl?.setReadWriteAccessForPublic()
publicDB.save(publicNote) { /* ... */ }
```

:::tips

**Read access**

Read access grants users right to *query* and *fetch* records, which includes getting all the fields of a record as well as the ACL of the record.


**Write access**

Write access grants users right to *save* and *delete* records, which includes adding,
updating and removing all the fields (**EXCEPT** [reserved columns][doc-reserved-columns]) of a record as well as the ACL of the record.

:::

## Record-based ACL By User

Similarly with ACL of Public, you can set the `NoAccessForUser`, `ReadOnlyForUser` and
`ReadWriteAccessForUser`) for each records.

Suppose you have three user records: `Tak`, `Benson` and `Rick`. And this is the security setting you want to apply:

- `Tak` has no access to the note.
- `Benson` can only read the note.
- `Rick` can both read and write the note.

```obj-c
SKYRecord *note = [SKYRecord recordWithRecordType:@"note"];
note[@"content"] = @"demo user acl";

[note.accessControl setNoAccessForUser:Tak];
[note.accessControl setReadOnlyForUser:Benson];
[note.accessControl setReadWriteAccessForUser:Rick];
```

```swift
let note = SKYRecord(recordType: "note")
note.setObject("demo user acl", forKey: "content" as NSCopying!)

note.accessControl?.setNoAccessForUser(Tak)
note.accessControl?.setReadOnlyForUser(Benson)
note.accessControl?.setReadWriteAccessForUser(Rick)
```

:::tips
The default ACL in Skygear is public read. So if you didn't assign any ACL to `Tak`, he will be able to read the note.
:::

## Record-based ACL By Role

Similar to ACL by User, suppose you have three roles: `Manager`, `Employee` and `Visitor`. (Learn how to set roles [here][doc-role-acl].)

```obj-c
SKYRecord *plan = [SKYRecord recordWithRecordType:@"plan"];
plan[@"content"] = @"future goals for company";

[plan.accessControl setNoAccessForRole:Visitor];
[plan.accessControl setReadOnlyForRole:Employee];
[plan.accessControl setReadWriteAccessForRole:Manager];
```

```swift
let plan = SKYRecord(recordType: "plan")
plan.setObject("future goals for company", forKey: "content" as NSCopying!)

plan.accessControl?.setNoAccessFor(Visitor)
plan.accessControl?.setReadOnlyFor(Employee)
plan.accessControl?.setReadWriteAccessFor(Manager)
```

## Working with User and Role

If you are defining the ACL of a record with both specific users and roles, and you want to check if a user has access to a record.

```obj-c
NSArray<SKYRecord *> *users = @[ /* user records */ ];
SKYRecord *record = /* record */;

[[[SKYContainer defaultContainer] auth] fetchRolesOfUsers:users completion:^(NSDictionary<NSString *,NSArray<SKYRole *> *> * _Nullable userRoles, NSError * _Nullable error) {
    SKYAccessControl *acl = record.accessControl;
    for (SKYRecord *user in users) {
        NSArray<SKYRole *> *userRole = userRoles[user.recordID.recordName];

        // check against the user record and user roles
    }
}];
```

```swift
let users: [SKYRecord] = [ /* user records */ ]
let record: SKYRecord = /* record */

SKYContainer.default().auth.fetchRoles(ofUsers: users) { (userRoles, error) in
    let acl = record.accessControl
    for user in users {
        let userRole = userRoles![user.recordID.recordName]

        // check against the user record and user roles
    }
}
```

## Set default ACL for a record type

Instead of setting ACL of each record object as mentioned above, you may set the default ACL of
all newly created records of a record type.

```obj-c
SKYAccessControl *acl = [SKYAccessControl emptyAccessControl];
[acl setReadOnlyForRole:Visitor];
[[[SKYContainer defaultContainer] publicCloudDatabase] defineDefaultAccessWithRecordType:@"note" access:acl completion:/* ... */];
```

```swift
let acl = SKYAccessControl.empty()
acl?.setReadOnlyFor(Visitor)
SKYContainer.default().publicCloudDatabase.defineDefaultAccess(withRecordType: "note", access: acl) { /* ... */ }
```

## SDK Default ACL

On top of setting Record Default ACL, you can also change the default ACL settings locally.
You may consider this as a convenient method of the SDK.

```obj-c
// giving admin role read write access to all new records
SKYRole *admin = [SKYRole roleWithName:@"manager"];
[acl setReadWriteAccessForRole:admin];
[[[SKYContainer defaultContainer] publicCloudDatabase] setDefaultAccessControl:acl];
```

```swift
// giving admin role read write access to all new records
let admin = SKYRole(name: "admin")
acl?.setReadWriteAccessFor(admin)
SKYContainer.default().publicCloudDatabase.defaultAccessControl = acl
```

After changing the default ACL setting, all records created in the future
will automatically have this ACL setting; however, ACL setting for existing
records created before this update will remain unchanged.

## Record Creation Access

When you call `save` on skygear cloud database, it may perform `create` or `update`.

However, ACL is stored in `_access` field of a record, and this does not work for `create`, since the record is not existed yet. Thus, Skygear provide another API for setting ACL for record creation.

```obj-c
[[[SKYContainer defaultContainer] publicCloudDatabase] defineCreationAccessWithRecordType:@"note" roles:@[Employee, Manager] completion:/* ... */];
[[[SKYContainer defaultContainer] publicCloudDatabase] defineCreationAccessWithRecordType:@"plan" roles:@[Manager] completion:/* ... */];
```

```swift
SKYContainer.default().publicCloudDatabase.defineCreationAccess(withRecordType: "note", roles: [Employee, Manager]) { /* ... */ }
SKYContainer.default().publicCloudDatabase.defineCreationAccess(withRecordType: "plan", roles: [Manager]) { /* ... */ }
```

## Example: An In-house Collaborative Editing Application

For example, you want to design the security model for an in-house collaborative
editing application. Each employee may join a group and create files, only
group member have access to the files of the group.

You can define a role for admin, and define default ACL.

```obj-c
SKYRole *admin = [SKYRole roleWithName:@"Admin"];

SKYAccessControl *acl = [SKYAccessControl emptyAccessControl];
[acl setNoAccessForPublic];
[acl setReadWriteAccessForRole:admin];
[[[SKYContainer defaultContainer] publicCloudDatabase] defineDefaultAccessWithRecordType:@"file" access:acl completion:/* ... */];
```

```swift
let admin = SKYRole(name: "Admin")

let acl = SKYAccessControl.empty()
acl?.setNoAccessForPublic()
acl?.setReadWriteAccessFor(admin)
SKYContainer.default().publicCloudDatabase.defineDefaultAccess(withRecordType: "file", access: acl) { /* ... */ }
```

You can define a role for each group.

```obj-c
NSString *groupID = @"...";
SKYRole *groupMember = [SKYRole roleWithName:[NSString stringWithFormat:@"group:%@:member", groupID]];
```

```swift
let groupID = "..."
let groupMember = SKYRole(name: "group:\(groupID):member")
```

When an employee want to join the group, the employee need to request
permission from the boss. Then the boss can assign the role to the employee.

```obj-c
NSArray<SKYRecord *> *someUsers = @[...];
[[[SKYContainer defaultContainer] auth] assignRoles:@[groupMember] toUsers:someUsers completion:/* ... */];
```

```swift
let someUsers: [SKYRecord] = [...]
SKYContainer.default().auth.assign([groupMember], toUsers: someUsers) { /* ... */ }
```

When a new file is created, you can set ACL of the file with the role, so
the new ACL will apply to the employees who have this role. Moreover, later
when a member leave the group, you may simply revoke the user role and the
user will no longer have access to the file. The ACL of file can remain
unchanged.

```obj-c
SKYRecord *group = [SKYRecord recordWithRecordType:@"group" name:groupID];
SKYReference *groupRef = [SKYReference referenceWithRecord:group];

SKYRecord *file = [SKYRecord recordWithRecordType:@"file"];
file[@"group"] = groupRef;
/* other attrs */

[file.accessControl setNoAccessForPublic];
[file.accessControl setReadWriteAccessForRole:groupMember];

[[[SKYContainer defaultContainer] publicCloudDatabase] saveRecord:file completion:/* ... */];
```

```swift
let group = SKYRecord(recordType: "group", name: groupID)
let groupRef = SKYReference(record: group)

let file = SKYRecord(recordType: "file")
file.setObject(groupRef, forKey: "group" as! NSCopying)

file.accessControl?.setNoAccessForPublic()
file.accessControl?.setReadWriteAccessFor(groupMember)

SKYContainer.default().publicCloudDatabase.save(file) { /* */ }
```

Now, you want to add a feature that a file can be shared to another employee
that is not in the group, but that employee cannot edit the file.

```obj-c
SKYRecord *aFile = /* a file record */;
SKYRecord *aUser = /* a user record */;

[aFile.accessControl setReadOnlyForUser:aUser];
[[[SKYContainer defaultContainer] publicCloudDatabase] saveRecord:aFile completion:/* ... */];
```

```swift
let aFile: SKYRecord = /* a file record */
let aUser: SKYRecord = /* a user record */

aFile.accessControl?.setReadOnlyForUser(aUser)
SKYContainer.default().publicCloudDatabase.save(file) { /* */ }
```

## Next Part
- [Field-based ACL][doc-field-acl]

[doc-role-acl]: /guides/cloud-db/acl-overview/ios/#acl-user-target
[doc-overview-acl]: /guides/cloud-db/acl-overview/ios/
[doc-field-acl]: /guides/cloud-db/field-acl/
