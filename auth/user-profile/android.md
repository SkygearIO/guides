---
title: User Record
---

[[toc]]


## Introduction

Whenever a new user signs up, a user record is automatically created for
you to track user information other than their username, email or other
user data.

The user record is created using the record type `user` with
the column `_id` storing the user ID, so you can use it
in the same way as using any other record types.
You can query and update a user record by manipulating using
the `user` record type.

Important note: This user record is created in the public database, i.e.
it is visible to any other user. Therefore you should not store any sensitive
information in this record type. You will need to create another record type
with the private database for information that can only be accessed by the
owner.

## Saving to the current user record

```java
Record user = new Record("user", skygear.getAuth().getCurrentUser().getId());
user.set("language", "en-US")
user.set("gender", "male")
user.set("age", 20)

Container skygear = Container.defaultContainer(this);
Database publicDB = skygear.getPublicDatabase();
publicDB.save(user, new RecordSaveResponseHandler() {
    @Override
    public void onSaveSuccess(Record user) {
        Log.i("Skygear User", "Saved");
    }

    @Override
    public void onSaveFail(Error error) {
        Log.i("Skygear User", "onSaveFail: Fail with reason:" + error.getMessage());
    }
})
```

## Retrieving the current user record

```java
Query userQuery = new Query("user")
        .equalTo("_id", skygear.getAuth().getCurrentUser().getId())

Container skygear = Container.defaultContainer(this);
Database publicDB = skygear.getPublicDatabase();
publicDB.query(userQuery, new RecordQueryResponseHandler() {
    @Override
    public void onQuerySuccess(Record[] records) {
        Log.i("User Query", String.format("Successfully got %d records", records.length));
    }

    @Override
    public void onQueryError(String reason) {
        Log.i("User Query", String.format("Fail with reason:%s", reason));
    }
});
```

<a id="search-users"></a>

## Retrieving another user record by email or username

You can retrieve the public records of other users by using their emails or
usernames. You can provide either a single email/username or an array of
emails/usernames.
The promise will be resolved by an array of matched user records.

```java
Query userQuery = new Query("user")
        .equalTo("email", "ben@oursky.com)

Container skygear = Container.defaultContainer(this);
Database publicDB = skygear.getPublicDatabase();
publicDB.query(userQuery, new RecordQueryResponseHandler() {
    @Override
    public void onQuerySuccess(Record[] records) {
        Log.i("User Query", String.format("Successfully got %d records", records.length));
    }

    @Override
    public void onQueryError(String reason) {
        Log.i("User Query", String.format("Fail with reason:%s", reason));
    }
});
```

```java
Query userQuery = new Query("user")
        .equalTo("username", "ben)

Container skygear = Container.defaultContainer(this);
Database publicDB = skygear.getPublicDatabase();
publicDB.query(userQuery, new RecordQueryResponseHandler() {
    @Override
    public void onQuerySuccess(Record[] records) {
        Log.i("User Query", String.format("Successfully got %d records", records.length));
    }

    @Override
    public void onQueryError(String reason) {
        Log.i("User Query", String.format("Fail with reason:%s", reason));
    }
});
```
