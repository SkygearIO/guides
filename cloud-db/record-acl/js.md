---
title: Record-based ACL
---

[[toc]]

Each record has a `_access` field, which stores the ACL of that record.
You may update the ACL of a record by calling API on the record object and
save it afterward.

## Record-based ACL By User

Suppose you have three user records: `Tak`, `Benson` and `Rick`.

```javascript
const Note = skygear.Record.extend('note');
const note = new Note({ content: 'demo user acl' });

note.setNoAccessForUser(Tak);
note.setReadOnlyForUser(Benson);
note.setReadWriteAccessForUser(Rick);
skygear.publicDB.save(note).then(...);

note.hasReadAccessForUser(Tak); // false
note.hasWriteAccessForUser(Benson); // false
```

## Record-based ACL By Role

Suppose you have three roles: `Manager`, `Employee` and `Visitor`.

```javascript
const Plan = skygear.Record.extend('plan');
const plan = new Plan({ title: 'future goals for company' });

plan.setNoAccessForRole(Visitor);
plan.setReadOnlyForRole(Employee);
plan.setReadWriteAccessForRole(Manager);
skygear.publicDB.save(plan).then(...);

plan.hasReadAccessForRole(Employee); // true
plan.hasWriteAccessForRole(Manager); // true
```

## Working with User and Role

If you are defining ACL with both user and role, and if you want to check if a
user has access to a record.

```javascript
const users = [ /* user records */ ];
const record = /* record */

skygear.auth.fetchUserRole(users).then((results) => {
  for (let user in users) {
    const userRoles = results[user._id];

    const hasReadAccess = record.access.hasReadAccess(user, userRoles);
    const hasWriteAccess = record.access.hasWriteAccess(user, userRoles);
  }
}, (error) => {
  console.log('Unable to fetch user roles', error);
});
```

## Record Default ACL

You may set the default ACL of a newly created record of a record type.

```javascript
const acl = new skygear.ACL();
acl.setReadOnlyForRole(Visitor);

skygear.publicDB.setRecordDefaultAccess(Note, acl);
```

## Record Creation Access

Record creation access is separate from write access for individual records.

```javascript
skygear.publicDB.setRecordCreateAccess(Note, [ Employee, Manager ]);
skygear.publicDB.setRecordCreateAccess(Plan, [ Manager ]);
```

## Example: A Collaborative Editing Application

For example, you want to design the security model for a collaborative editing
application. Each user may join a group and create files, only group member have
access to the files of the group.

You can define a role for each group, and define default ACL.

```javascript
const File = skygear.Record.extend('file');
const Group = skygear.Record.extend('group');

const Admin = new skygear.Role('admin')

const acl = new skygear.ACL();
acl.setPublicNoAccess();
acl.setReadWriteAccessForRole(Admin);

skygear.publicDB.setRecordDefaultAccess(File, acl);
```

You can define a role for each group.

```javascript
const groupID = '...';
const GroupMember = new skygear.Role(`group:${groupID}:member`);
```

When new users join the group, you can assign the role to the user.

```javascript
const someUsers = [...];
skygear.auth.assignUserRole(someUsers, [GroupMember]).then(...);
```

When a new file is created, you can set ACL of the file with the role, so
the new ACL will apply to the users who have this role. Moreover, later when
a user leave the group, you may simply revoke the user role and the user will
no longer have access to the file. The ACL of file can remain unchanged.

```javascript
// create a referece of an existing group record
const group = new Group({ _id: `group/${groupID}` });
const groupRef = new Skygear.Reference(group);

// create a new file object
const file = new File({
  group: group,
  ... // other attrs
});

// define access rights of the file
file.setPublicNoAccess();
file.setReadWriteAccessForRole(GroupMember);

skygear.publicDB.save(file).then(...);
```

Now, you want to add a feature that a file can be shared to another user that
is not in the group, but that user cannot edit the file.

```javascript
const aFile = /* a file record */;
const aUser = /* a user record */;

aFile.setReadOnlyForUser(aUser);
skygear.publicDB.save(aFile).then(...);
```
