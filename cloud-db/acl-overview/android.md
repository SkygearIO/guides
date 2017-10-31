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

If you want to customize these security settings, Skygear supports access control on records. You can use it to control whether a user can query, update or delete a record.

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

:::tips
When you should use Record-based ACL or Field-based ACL?

If every single record has the same ACL, that the same  have the same access rights to a record, you can use Record-based ACL.

Otherwise, if a set of users have has different access rights for each field of a record, you would need to use Field-based ACL.
:::

Before diving into Record-based ACL and Field-based ACL, let's learn about the basic concepts of Skygear ACL.

## ACL Resource

### Record-based

Resource is defined in each record. Thus the ACL affects the record itself.

### Field-based

Resource in is specified by record type and record field.

You may find more details about resource matching in [Field-based ACL][doc-field-acl-matching].

## ACL Access Level

There are 3 access levels in Skygear ACL:

- No Access
- Read Write Access
- Read Only Access

This is how you can set it:

```java
AccessControl acl = new AccessControl();

acl.addEntry(new AccessControl.Entry(AccessControl.Level.NO_ACCESS));
acl.addEntry(new AccessControl.Entry(AccessControl.Level.READ_ONLY));
acl.addEntry(new AccessControl.Entry(AccessControl.Level.READ_WRITE));
```

:::tips

**Read access**

Read access grants users right to *query* and *fetch* records, which includes getting all the fields of a record as well as the ACL of the record.


**Write access**

Write access grants users right to *save* and *delete* records, which includes adding,
updating and removing all the fields (**EXCEPT** [reserved columns][doc-reserved-columns]) of a record as well as the ACL of the record.

:::

## ACL User Target

User Target defines *who* can perform the specified actions. It can be a specific user or a group of users. Below are the types of User Target you can define.

- **Public**: refers to all other users of the app, unauthenticated (not logged in) users included
- **By User**: users specified by User ID
- **By Role**: users grouped into specific roles by admin

Below we will talk about how to create roles and assign a role to users.

### Role Configuration

#### Define role

You can create different roles and later use them to design ACL targeting users of specific roles.

```java
Role manager = new Role("manager");
Role visitor = new Role("visitor");
```

#### Admin role

In Skygear roles are either admin role or non-admin role. By default, there is an admin role called `Admin`.

What only admin role can do:

- Assign role to user
- Revoke role to user

:::tips
For security concern, only admin can change the role of a user, thus the user must request another user with admin role to change the role for him.

If a group is just used for display purpose, like group member. Then the group is a public group. Users actually do not need permission from Admin to join the group.

Do NOT overload user roles to model group if no access control is necessary. It will have performance penalty since it involves an extra check during query/save.
:::


#### Setting admin role and default role in development

:::caution
Admin role and default role must be set during development. The two operations are banned in production environment.
:::

You define new roles to be admin role.

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

You can also set the default roles so that newly signed up user would start with these roles.

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

## Next Part
- [Record-based ACL][doc-record-acl]

[wiki-acl]: https://en.wikipedia.org/wiki/Access_control_list
[doc-reserved-columns]: /guides/cloud-db/basics/android/#reserved-columns
[doc-record-acl]: /guides/cloud-db/record-acl/android/
[doc-field-acl]: /guides/cloud-db/field-acl/
[doc-field-acl-matching]: /guides/cloud-db/field-acl/#resource-matching
