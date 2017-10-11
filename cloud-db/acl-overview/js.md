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

```javascript
// not logged in as any user
const Note = skygear.Record.extend('note');
const noteQuery = new skygear.Query(Note);

skygear.publicDB.query(noteQuery).then((records) => {
  console.log('Successful to query Note records');

  const note = records[0];
  note.content = 'Hello World!';
  return skygear.publicDB.save(note);
}).then((records) => {
  // You will NOT succeed
}, (error) => {
  // You will get not Authenticated error here
  console.log(error);
});
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

```javascript
// either way of defining roles is fine
const Manager = skygear.Role.define('manager');
const Visitor = new skygear.Role('visitor');
```

You can set certain roles as admin so that they have the power to change other
users' roles.

```javascript
skygear.auth.setAdminRole([Manager]).then((roles) => {
  console.log(roles); // [ 'manager' ]
}, (error) => {
  console.error(error);
});
```

You can also set the default roles so that newly signed up user would start
with these roles.

```javascript
skygear.auth.setDefaultRole([Visitor]).then(...);
```

#### Assign and Revoke Roles

For admin user to change roles of other users:

```javascript
// You must login as an admin user
const targetUsername = ['John', 'Ken'];
const userQuery = new skygear.Query(skygear.UserRecord);
userQuery.contains('username', targetUsername);
skygear.publicDB.query(userQuery).then((records) => {
  if (records.length != targetUsername.length) {
    console.log('Cannot find both John and Ken');
  }

  return Promise.all([
    skygear.auth.revokeUserRole(records, [Visitor]),
    skygear.auth.assignUserRole(records, [Manager]),
  ]);
}).then(() => {
  console.log('John and Ken are promoted from Visitor to Manager');
}, () => {
  console.log('You are not admin user');
});
```

## Changing Default ACL Settings

You can change the default ACL settings:

```javascript
const acl = new skygear.ACL();
// choose one setting among the three
acl.setPublicNoAccess();
acl.setPublicReadOnly(); // default
acl.setPublicReadWriteAccess();
skygear.publicDB.setDefaultACL(acl); // will change skygear.defaultACL

skygear.publicDB.defaultACL.hasPublicReadAccess(); // default true
skygear.publicDB.defaultACL.hasPublicWriteAccess(); // default false

// giving admin role read write access to all new records
const Admin = new skygear.Role('admin');
acl.setReadWriteAccessForRole(Admin);
skygear.publicDB.setDefaultACL(acl);
```

After changing the default ACL setting, all records created in the future
will automatically have this ACL setting; however, ACL setting for existing
records created before this update will remain unchanged.

[doc-reserved-columns]: /guides/cloud-db/basics/js/#reserved-columns
