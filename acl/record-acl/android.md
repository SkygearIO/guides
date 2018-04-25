---
title: Record-based ACL
---

[[toc]]

Record-based ACL is useful when users have different access to each record.

Take Google doc as an example, you can only access a doc (a record) when the right is granted to you by the owner.

In Skygear, each record has a `_access` field, which stores the ACL of that record. You may update the ACL of a record by calling ACL API on the record and save it afterward.

Now let's have a look at how to set Record-based ACL.

:::tips
To understand record-based ACL you need basic Skygear ACL concepts. Learn about it [here][doc-overview-acl] if you haven't.
:::

## Record-based ACL of Public

Public ACL (including `PublicNoAccess`, `PublicReadOnly` and
`PublicReadWriteAccess`) define the permission unauthenticated users have. You can change the record
ACL for Public users as follows:

```java
Record secretNote = new Record("note");
secretNote.set("content", "I am your father");
secretNote.setPublicNoAccess();
skygear.getPublicDatabase().save(secretNote, /* ... */);

Record publicNote = new Record("note");
publicNote.set("content", "Hello world!");
publicNote.setPublicReadWrite();
skygear.getPublicDatabase().save(publicNote, /* ... */);
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

```java
Record note = new Record("note");
note.set("content", "demo user acl");

note.setNoAccess(Tak);
note.setReadOnly(Benson);
note.setReadWriteAccess(Rick);
skygear.getPublicDatabase().save(note, /* ... */);
```

:::tips
The default ACL in Skygear is public read. So if you didn't assign any ACL to `Tak`, he will be able to read the note.
:::

## Record-based ACL By Role

Similar to ACL by User, suppose you have three roles: `Manager`, `Employee` and `Visitor`. (Learn how to set roles [here][doc-role-acl].)

```java
Record plan = new Record("plan");
plan.set("title", "future goals for company");

plan.setNoAccess(Visitor);
plan.setReadOnly(Employee);
plan.setReadWriteAccess(Manager);
skygear.getPublicDatabase().save(plan, /* ... */);
```

## Working with User and Role

If you are defining the ACL of a record with both specific users and roles, and you want to check if a user has access to a record.

```java
Record[] users = new Record[]{ /* user records */ };
Record record = /* record */;

skygear.getAuth().fetchUserRole(users, new FetchUserRoleResponseHandler(){
    @Override
    public void onFetchSuccess(Map<String, Role[]> userRoles) {
        for (Record user : users) {
            Role[] userRole = userRoles.get(user.getId());

            // check against the user record and user roles
        }
    }

    @Override
    public void onFetchFail(Error error) {

    }
});
```

## Set default ACL for a record type

Coming Soon

## SDK Default ACL

On top of setting Record Default ACL, you can also change the default ACL settings locally.
You may consider this as a convenient method of the SDK.

```java
Container skygear = Container.defaultContainer(this);
AccessControl acl = /* construct your own access control */

skygear.setDefaultAccessControl(acl);
```

After changing the default ACL setting, all records created in the future
will automatically have this ACL setting; however, ACL setting for existing
records created before this update will remain unchanged.

## Record Creation Access

Coming Soon

## Next Part
- [Field-based ACL][doc-field-acl]

[doc-role-acl]: /guides/cloud-db/acl-overview/js/#acl-user-target
[doc-overview-acl]: /guides/cloud-db/acl-overview/js/
[doc-field-acl]: /guides/cloud-db/field-acl/
