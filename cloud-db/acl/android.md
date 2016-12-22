---
title: Access Control (ACL)
---

<a name='acl-type'></a>
## Access Type

Skygear supports access control on records. You can use it to control whether
a user can query, update or delete a record.

Access Control in Skygear can be set as 3 levels:

1. No Access
2. Read Only Access
3. Read Write Access

The access control in Skygear can be divided in following type:

- [Public Access Control][doc-acl-public]
- [User-based Access Control][doc-acl-user-based]
- [Role-based Access Control][doc-acl-role-based]

<a name='acl-default'></a>
## Default ACL Settings

Besides of setting each record to a specific access control, Skygear also
allows you set a default access control for records. If you don't set any
default access control, it would be **public readable**.

```java

Container skygear = Container.defaultContainer(this);
AccessControl acl = /* construct your own access control */

skygear.setDefaultAccessControl(acl);

```

<a id='acl-public'></a>
## Public ACL
To control the public access of a record, you can set the public access control
of a record:

```java
Container skygear = Container.defaultContainer(this);
Record aRecord = /* Create or Query a record */

aRecord.setPublicNoAccess();  // for public no access
aRecord.setPublicReadOnly();  // for public read-only access
aRecord.setPublicReadWrite(); // for public read-write access

skygear.getPublicDatabase().save(aRecord, new RecordSaveResponseHandler(){
    // ...
});

```

Of course, you can check whether a record is publicly readable / writable as
following:

```java
Record aRecord = /* Create or Query a record */

boolean isPublicReadable = aRecord.isPublicReadable();
boolean isPublicWritable = aRecord.isPublicWritable();

```

<a id='acl-user-based'></a>
## ACL by User

User-based access control is to set the access level of a record for a specific
user.

To set the user-based access control, you may first need query the user:

```java
Container skygear = Container.defaultContainer(this);
skygear.getUserByEmail("user01@skygear.dev", new UserQueryResponseHandler(){
    @Override
    public void onQuerySuccess(User[] users) {
        User theUser = users[0];
        // your logic to set the user-based access control
    }

    @Override
    public void onQueryFail(String reason) {
        // Error handling...
    }
});
```

After getting the user, you can set the access control of a record for that user.

```java
Container skygear = Container.defaultContainer(this);
User theUser = /* Query a user */
Record aRecord = /* Create or Query a record */

aRecord.setNoAccess(theUser);        // no access for the user
aRecord.setReadOnly(theUser);        // read-only access for the user
aRecord.setReadWriteAccess(theUser); // read-write access for the user

skygear.getPublicDatabase().save(aRecord, new RecordSaveResponseHandler(){
    // ...
});

```

You can check whether a record is readable / writable for a specific user:

```java
User theUser = /* Query a user */
Record aRecord = /* Create or Query a record */

boolean isReadable = aRecord.isReadable(theUser);
boolean isWritable = aRecord.isWritable(theUser);

```

If you only get the user ID, you can also set user-based access control of
a record:

```java
Container skygear = Container.defaultContainer(this);
String userId = /* Get the User ID */
Record aRecord = /* Create or Query a record */

aRecord.setNoAccess(userId);        // no access for the user
aRecord.setReadOnly(userId);        // read-only access for the user
aRecord.setReadWriteAccess(userId); // read-write access for the user

skygear.getPublicDatabase().save(aRecord, new RecordSaveResponseHandler(){
    // ...
});

```

<a id='acl-role-based'></a>
## ACL by User Role

Skygear supports assigning a role to a user in order to control the access of
a record for a group of users.

```java
Container skygear = Container.defaultContainer(this);
User theUser = /* Query a user */
Role programmer = new Role("Programmer");
Role projectManager = new Role("ProjectManager");

theUser.addRole(programmer);
theUser.removeRole(projectManager);

skygear.saveUser(theUser, new UserSaveResponseHandler() {
    @Override
    public void onSaveSuccess(User user) {
        // The roles of the user has been updated
    }

    @Override
    public void onSaveFail(String reason) {
        // Error handling...
    }
});

```

Please be reminded that assigning roles to user requires the current user is of
an admin role.

To setting roles as an admin role, you can:

```java
Container skygear = Container.defaultContainer(this);
Role admin = new Role("Admin");
Role boss = new Role("Boss");

skygear.setAdminRole(new Role[]{ admin, boss }, new SetRoleResponseHandler() {
    @Override
    public void onSetSuccess(Role[] roles) {
        // Admin roles are updated
    }

    @Override
    public void onSetFail(String reason) {
        // Error handling...
    }
});

```

You can check whether a user has a certain role:

```java
User theUser = /* Query a user */
Role programmer = new Role("Programmer");

boolean isProgrammer = theUser.hasRole(programmer);

```

After managing user using role, you can set the role-based access control as
following:

```java
Role programmer = new Role("Programmer");

Container skygear = Container.defaultContainer(this);
Record aRecord = /* Create or Query a record */

aRecord.setNoAccess(programmer);        // no access for the role
aRecord.setReadOnly(programmer);        // read-only access for the role
aRecord.setReadWriteAccess(programmer); // read-write access for the role

skygear.getPublicDatabase().save(aRecord, new RecordSaveResponseHandler(){
    // ...
});

```

You can check whether a record is readable / writable for a specific role:

```java
Role programmer = new Role("Programmer");
Record aRecord = /* Create or Query a record */

boolean isReadable = aRecord.isReadable(programmer);
boolean isWritable = aRecord.isWritable(programmer);

```

<a name='acl-record'></a>
## ACL by Record Type

[doc-acl-public]: #acl-public
[doc-acl-user-based]: #acl-user-based
[doc-acl-role-based]: #acl-role-based
