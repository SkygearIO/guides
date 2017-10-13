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

You can change the record ACL by calling the ACL API on the record and saving it afterwards.

```java
// this will be the same as saving the record to privateDB
Record secretNote = new Record("note");
secretNote.set("content", "I am your father");
secretNote.setPublicNoAccess();
skygear.getPublicDatabase().save(secretNote, /* ... */);

Record publicNote = new Record("note");
publicNote.set("content", "Hello world!");
publicNote.setPublicReadWrite();
skygear.getPublicDatabase().save(publicNote, /* ... */);
```

## Record-based ACL By User

Suppose you have three user records: `Tak`, `Benson` and `Rick`. And this is the security setting you want to apply:

- Tak has no access to the note.
- Benson can only read the note.
- Rick can both read and write the note.

```java
Record note = new Record("note");
note.set("content", "demo user acl");

note.setNoAccess(Tak);
note.setReadOnly(Benson);
note.setReadWriteAccess(Rick);
skygear.getPublicDatabase().save(note, /* ... */);
```

:::tips
The default ACL in Skygear is public read. So if you didn't assign any ACL to Tak, he will be able to read the note.
:::

## Record-based ACL By Role

Suppose you have three roles: `Manager`, `Employee` and `Visitor`. (Learn how to set roles [here][doc-role-acl].)

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

## Record Default ACL

Coming Soon

## Record Creation Access

Coming Soon

## Next Part
- [Field-based ACL][doc-field-acl]

[doc-role-acl]: /guides/cloud-db/acl-overview/js/#acl-user-target
[doc-overview-acl]: /guides/cloud-db/acl-overview/js/
[doc-field-acl]: /guides/cloud-db/field-acl/
