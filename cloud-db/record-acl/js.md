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

You can change the record ACL by calling the ACL API on the record and saving it afterwards.

```javascript
const secretNote = new Note({ content: 'I am your father' });
const publicNote = new Note({ content: 'Hello world!' });

// this will be the same as saving the record to privateDB
secretNote.setPublicNoAccess();
skygear.publicDB.save(secretNote);

publicNote.setPublicReadWriteAccess();
skygear.publicDB.save(publicNote);
```

## Record-based ACL By User

Suppose you have three user records: `Tak`, `Benson` and `Rick`. And this is the security setting you want to apply:

- `Tak` has no access to the note.
- `Benson` can only read the note.
- `Rick` can both read and write the note.

```javascript
const Note = skygear.Record.extend('note');
const note = new Note({ content: 'demo user acl' });

note.setNoAccessForUser(Tak);
note.setReadOnlyForUser(Benson);
note.setReadWriteAccessForUser(Rick);
skygear.publicDB.save(note).then(...);

// check if users have read access or write access
note.hasReadAccessForUser(Tak); // false
note.hasWriteAccessForUser(Benson); // false
```

:::tips
The default ACL in Skygear is public read. So if you didn't assign any ACL to `Tak`, he will be able to read the note.
:::

## Record-based ACL By Role

Suppose you have three roles: `Manager`, `Employee` and `Visitor`. (Learn how to set roles [here][doc-role-acl].)

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

If you are defining the ACL of a record with both specific users and roles, and you want to check if a user has access to a record.

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

When you call `save` on skygear cloud database, it may perform `create` or `update`.

However, ACL is stored in `_access` field of a record, and this does not work for `create`, since the record is not existed yet. Thus, Skygear provide another API for setting ACL for record creation.

```javascript
skygear.publicDB.setRecordCreateAccess(Note, [ Employee, Manager ]);
skygear.publicDB.setRecordCreateAccess(Plan, [ Manager ]);

// logged in as Employee
const newPlan = new Plan();
skygear.publicDB.save(newPlan).then(() => {
    // You will NOT succeed
}, () => {
    // You will get error here
});
```

## Example: An In-house Collaborative Editing Application

For example, you want to design the security model for an in-house collaborative
editing application. Each employee may join a group and create files, only
group member have access to the files of the group.

You can define a role for admin, and define default ACL.

```javascript
const File = skygear.Record.extend('file');
const Group = skygear.Record.extend('group');

const Admin = new skygear.Role('Admin')

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

When an employee want to join the group, the employee need to request
permission from the boss. Then the boss can assign the role to the employee.

```javascript
// by the boss
const someUsers = [...];
skygear.auth.assignUserRole(someUsers, [GroupMember]).then(...);
```

When a new file is created, you can set ACL of the file with the role, so
the new ACL will apply to the employees who have this role. Moreover, later
when a member leave the group, you may simply revoke the user role and the
user will no longer have access to the file. The ACL of file can remain
unchanged.

```javascript
// create a referece of an existing group record
const group = new Group({ _id: `group/${groupID}` });
const groupRef = new Skygear.Reference(group);

// create a new file object
const file = new File({
  group: groupRef,
  ... // other attrs
});

// define access rights of the file
file.setPublicNoAccess();
file.setReadWriteAccessForRole(GroupMember);

skygear.publicDB.save(file).then(...);
```

Now, you want to add a feature that a file can be shared to another employee
that is not in the group, but that employee cannot edit the file.

```javascript
const aFile = /* a file record */;
const aUser = /* a user record */;

aFile.setReadOnlyForUser(aUser);
skygear.publicDB.save(aFile).then(...);
```

## Next Part
- [Field-based ACL][doc-field-acl]

[doc-role-acl]: /guides/cloud-db/acl-overview/js/#acl-user-target
[doc-overview-acl]: /guides/cloud-db/acl-overview/js/
[doc-field-acl]: /guides/cloud-db/field-acl/
