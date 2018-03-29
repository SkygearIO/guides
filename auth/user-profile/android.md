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
the `user` record type. Note that the `_id` provided while creating a new user
record has to be unique - i.e. `_id` must be not be used to create another user before.

Important note: This user record is created in the public database, i.e.
it is visible to any other user. Therefore you should not store any sensitive
information in this record type. You will need to create another record type
with the private database for information that can only be accessed by the
owner.

## Saving to the current user record

```java
Container skygear = Container.defaultContainer(this);

Record user = new Record("user", "new-user-id");
user.set("username", "new-username");
user.set("language", "en-US");
user.set("gender", "male");
user.set("age", 20);

Database publicDB = skygear.getPublicDatabase();
publicDB.save(user, new RecordSaveResponseHandler() {
  @Override
  public void onSaveSuccess(Record[] records) {
    for (int i = 0; i < records.length; i++) {
      Log.i("Skygear User", "Saved user with username: " + records[i].get("username"));
    }
  }

  @Override
  public void onPartiallySaveSuccess(Map<String, Record> successRecords, Map<String, Error> errors) {
    for (Map.Entry<String, Record> entry : successRecords.entrySet()) {
      Log.i("Skygear User", "Saved user with username: " + entry.getValue().get("username"));
    }

    for (Map.Entry<String, Error> entry: errors.entrySet()) {
      Log.e("Skygear User", "Save error - reason: " + entry.getValue().getMessage());
    }
  }

  @Override
  public void onSaveFail(Error error) {
    Log.e("Skygear User", "onSaveFail: Fail with reason:" + error.getMessage());
  }
});
```
```kotlin
val skygear = Container.defaultContainer(this)

val user = Record("user", "new-user-id")
user.set("username", "new-username")
user.set("language", "en-US")
user.set("gender", "male")
user.set("age", 20)

val publicDB = skygear.publicDatabase
publicDB.save(user, object : RecordSaveResponseHandler() {
  override fun onSaveSuccess(records: Array<Record>) {
    records.forEach { user ->
      Log.i("Skygear User", "Saved user with username: " + user.get("username"))
    }
  }

  override fun onPartiallySaveSuccess(successRecords: Map<String, Record>, errors: Map<String, Error>) {
    successRecords.forEach { (_, record) ->
      Log.i("Skygear User", "Saved user with username: ${record.get("username")}")
    }

    errors.forEach { (_, error) ->
      Log.e("Skygear User", "Save error - reason: ${error.message}")
    }
  }

  override fun onSaveFail(error: Error) {
    Log.e("Skygear User", "onSaveFail: Fail with reason:" + error.message)
  }
})
```

## Retrieving the current user record

```java
Container skygear = Container.defaultContainer(this);

Query userQuery = new Query("user")
  .equalTo("_id", skygear.getAuth().getCurrentUser().getId());

Database publicDB = skygear.getPublicDatabase();
publicDB.query(userQuery, new RecordQueryResponseHandler() {
  @Override
  public void onQuerySuccess(Record[] records) {
    Log.i("User Query", String.format("Successfully retrieved %d records", records.length));
  }

  @Override
  public void onQueryError(Error error) {
    Log.e("User Query", String.format("Fail with reason: " + error.getMessage()));
  }
});
```
```kotlin
val skygear = Container.defaultContainer(this)

val userQuery = Query("user").equalTo("_id", skygear.auth.currentUser.id)

val publicDB = skygear.publicDatabase
publicDB.query(userQuery, object : RecordQueryResponseHandler() {
  override fun onQuerySuccess(records: Array<out Record>) {
    Log.i("User Query", "Successfully retrieved ${records.size} records")
  }

  override fun onQueryError(error: Error) {
    Log.e("User Query", "Fail with reason: ${error.message}")
  }
})
```

<a id="search-users"></a>

## Retrieving another user record by email or username

You can retrieve the public records of other users by using their emails or
usernames. You can provide either a single email/username or an array of
emails/usernames.
The promise will be resolved by an array of matched user records.

```java
Container skygear = Container.defaultContainer(this);

Query userQuery = new Query("user")
  .equalTo("email", "ben@oursky.com");

Database publicDB = skygear.getPublicDatabase();
publicDB.query(userQuery, new RecordQueryResponseHandler() {
  @Override
  public void onQuerySuccess(Record[] records) {
    Log.i("User Query", String.format("Successfully retrieved %d records", records.length));
  }

  @Override
  public void onQueryError(Error error) {
    Log.e("User Query", String.format("Fail with reason: ", error.getMessage()));
  }
});
```
```kotlin
val skygear = Container.defaultContainer(this)

val userQuery = Query("user").equalTo("email", "ben@oursky.com")

val publicDB = skygear.publicDatabase
publicDB.query(userQuery, object : RecordQueryResponseHandler() {
  override fun onQuerySuccess(records: Array<out Record>) {
    Log.i("User Query", "Successfully retrieved ${records.size} records")
  }

  override fun onQueryError(error: Error) {
    Log.e("User Query", "Fail with reason: ${error.message}")
  }
})
```

```java
Container skygear = Container.defaultContainer(this);

Query userQuery = new Query("user")
  .equalTo("username", "ben");

Database publicDB = skygear.getPublicDatabase();
publicDB.query(userQuery, new RecordQueryResponseHandler() {
  @Override
  public void onQuerySuccess(Record[] records) {
    Log.i("User Query", String.format("Successfully retrieved %d records", records.length));
  }

  @Override
  public void onQueryError(Error error) {
    Log.e("User Query", String.format("Fail with reason: " + error.getMessage()));
  }
});
```
```kotlin
val skygear = Container.defaultContainer(this)

val userQuery = Query("user").equalTo("email", "ben")

val publicDB = skygear.publicDatabase
publicDB.query(userQuery, object : RecordQueryResponseHandler() {
  override fun onQuerySuccess(records: Array<out Record>) {
    Log.i("User Query", "Successfully retrieved ${records.size} records")
  }

  override fun onQueryError(error: Error) {
    Log.e("User Query", "Fail with reason: ${error.message}")
  }
})
```
