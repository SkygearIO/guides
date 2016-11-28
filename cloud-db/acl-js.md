---
title: Access Control (ACL)
---

<a name='acl-type'></a>
## Access Type

There are two types of access for each record: read and write.
- Read access allows querying and fetching records, which includes getting
  all the fields of records as well as the access control settings.
- Write access allows saving and deleting records, which includes adding,
  updating and removing all fields (**EXCEPT**
  [reserved columns](/js/guide/record#reserved)) of records
  as well as access control settings for the record.

However, even with write access, ownership of records cannot be changed.

<a name='acl-default'></a>
## Default ACL Settings

By default, read access is granted to the public and write access is granted
only to owner of the record. Here public refers to all other users and
unauthenticated (not logged in) users. You can change this behavior as well:

``` javascript
const acl = new skygear.ACL();
// choose one setting among the three
acl.setPublicNoAccess();
acl.setPublicReadOnly(); // default
acl.setPublicReadWriteAccess();
skygear.setDefaultACL(acl); // will change skygear.defaultACL

skygear.defaultACL.hasPublicReadAccess(); // default true
skygear.defaultACL.hasPublicWriteAccess(); // default false
```

After changing the default ACL setting, all records created in the future
will automatically have this ACL setting; however, ACL setting for existing
records created before this update will remain unchanged.

<a name='acl-user'></a>
## ACL by User

Suppose you have three user objects: `Tak`, `Benson` and `Rick`.

``` javascript
const Note = skygear.Record.extend('note');
const note = new Note({ content: 'demo user acl' });

note.setNoAccessForUser(Tak);
note.setReadOnlyForUser(Benson);
note.setReadWriteAccessForUser(Rick);
skygear.publicDB.save(note).then(...);

note.hasReadAccessForUser(Tak); // false
note.hasWriteAccessForUser(Benson); // false
```

<a name='acl-role'></a>
## ACL by User Role

### Defining roles

You can create different roles and later use them to design role-based access
control. You can set certain roles as admin so that they have the power to
change other users' roles. You can also set the default roles so that newly
signed up user would start with these roles.

``` javascript
const Manager = skygear.Role.define('manager');
const Visitor = new skygear.Role('visitor');
// either way of defining roles is fine
skygear.setAdminRole([Manager]).then((roles) => {
  console.log(roles); // [ 'manager' ]
}, (error) => {
  console.error(error);
});
skygear.setDefaultRole([Visitor]).then(...);
```

### Changing roles

For admin user to change roles of other users:

``` javascript
// skygear.currentUser should be an admin user
skygear.getUsersByEmail(['johndoe@example.com']).then((records) => {
  const john = records[0];
  john.addRole(Manager);
  john.removeRole(Visitor);
  return skygear.saveUser(john);
}, (error) => {
  console.error('John is not found')
}).then((john) => {
  console.log('John is promoted from Visitor to Manager');
}, (error) => {
  console.error('You are not admin user');
});
```

### Role-based access control per record

Suppose you have three roles: `Manager`, `Employee` and `Visitor`.

``` javascript
const Plan = skygear.Record.extend('plan');
const plan = new Plan({ title: 'future goals for company' });

plan.setNoAccessForRole(Visitor);
plan.setReadOnlyForRole(Employee);
plan.setReadWriteAccessForRole(Manager);
skygear.publicDB.save(plan).then(...);

plan.hasReadAccessForRole(Employee); // true
plan.hasWriteAccessForRole(Manager); // true
```

<a name='acl-record'></a>
## ACL by Record Type

### Record creation access control

Record creation access is separate from write access for individual records:

``` javascript
skygear.setRecordCreateAccess(Note, [ Employee, Manager ]);
skygear.setRecordCreateAccess(Plan, [ Manager ]);
```
