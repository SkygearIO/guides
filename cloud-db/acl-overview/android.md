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

```java
// not logged in as any user
Query noteQuery = new Query("note");
Database publicDB = skygear.getPublicDatabase();

publicDB.query(noteQuery, new RecordQueryResponseHandler() {
    @Override
    public void onQuerySuccess(Record[] records) {
        // Successful to query Note records
        Log.i("ACL", String.format("Successfully got %d records", records.length));
        Record aNote = records[0];
        aNote.set("content", "Hello World!");
        publicDB.save(aNote, new RecordSaveResponseHandler(){
            @Override
            public void onSaveSuccess(Record[] records) {
            }

            @Override
            public void onPartiallySaveSuccess(
                Map<String, Record> successRecords,
                Map<String, String> reasons
            ) {
            }

            @Override
            public void onSaveFail(String reason) {
              // Fail for not authenticated user
              Log.i("ACL", String.format("Fail with reason: %s", reason));
            }
        });
    }

    @Override
    public void onQueryError(String reason) {
    }
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

```java
Role manager = new Role("manager");
Role visitor = new Role("visitor");
```

You can set certain roles as admin so that they have the power to change other
users' roles.

```java
skygear.getAuth().setAdminRole(new Role[]{ manager }, new SetRoleResponseHandler() {
    @Override
    public void onSetSuccess(Role[] roles) {
        Log.i("ACL", "Admin roles are updated");
    }

    @Override
    public void onSetFail(String reason) {
        Log.i("ACL", String.format("Fail with reason: %s", reason));
    }
});
```

You can also set the default roles so that newly signed up user would start
with these roles.

```java
skygear.getAuth().setDefaultRole(new Role[]{ visitor }, new SetRoleResponseHandler() {
    @Override
    public void onSetSuccess(Role[] roles) {
        Log.i("ACL", "Default roles are updated");
    }

    @Override
    public void onSetFail(String reason) {
        Log.i("ACL", String.format("Fail with reason: %s", reason));
    }
});
```

#### Assign and Revoke Roles

For admin user to change roles of other users:

```java
// You must login as an admin user
SetUserRoleResponseHandler assignHandler = new SetUserRoleResponseHandler() {
    @Override
    public void onSetSuccess() {
        Log.i("ACL", "Roles are assigned");
    }

    @Override
    public void onSetFail(String reason) {
        Log.i("ACL", String.format("Fail with reason: %s", reason));
    }
};

String[] targetUserIDs = new String[]{ "John", "Ken" };
String[] rolesToRevoke = new Role[]{ "visitor" };
String[] rolesToAssign = new Role[]{ "manager" };
skygear.getAuth().revokeUserRole(targetUserIDs, rolesToRevoke, new SetUserRoleResponseHandler() {
    @Override
    public void onSetSuccess() {
        Log.i("ACL", "Roles are revoked");
        skygear.getAuth().assignUserRole(targetUserIDs, rolesToAssign, assignHandler);
    }

    @Override
    public void onSetFail(String reason) {
        Log.i("ACL", String.format("Fail with reason: %s", reason));
    }
});
```

## Changing Default ACL Settings

You can change the default ACL settings:

```java
Container skygear = Container.defaultContainer(this);
AccessControl acl = /* construct your own access control */

skygear.setDefaultAccessControl(acl);
```

After changing the default ACL setting, all records created in the future
will automatically have this ACL setting; however, ACL setting for existing
records created before this update will remain unchanged.

[doc-reserved-columns]: /guides/cloud-db/basics/js/#reserved-columns
