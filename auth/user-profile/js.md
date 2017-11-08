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

``` javascript
// skygear.UserRecord is equivalent to skygear.Record.extend('user')
const modifiedRecord = new skygear.UserRecord({
  '_id': 'user/' + skygear.auth.currentUser.id,
  'language': 'en-US',
  'gender': 'male',
  'age': 20,
});
skygear.publicDB.save(modifiedRecord).then((record) => {
  console.log(record); // updated user record
});
```

## Retrieving the current user record

``` javascript
const query = new skygear.Query(skygear.UserRecord);
query.equalTo('_id', skygear.auth.currentUser.id);
skygear.publicDB.query(query).then((records) => {
  const record = records[0];
  console.log(record);
}, (error) => {
  console.error(error);
});
```

<a id="search-users"></a>

## Retrieving another user record by email or username

You can retrieve the public records of other users by using their emails or
usernames. You can provide either a single email/username or an array of
emails/usernames.
The promise will be resolved by an array of matched user records.

``` javascript
// you can also pass an array of emails
const query = new skygear.Query(skygear.UserRecord);
query.equalTo('email', 'ben@oursky.com');
skygear.publicDB.query(query).then((records) => {
  const record = records[0];
  console.log(record);
}, (error) => {
  console.error(error);
});
```

``` javascript
// you can also pass an array of usernames
const query = new skygear.Query(skygear.UserRecord);
query.equalTo('username', 'ben');
skygear.publicDB.query(query).then((records) => {
  const record = records[0];
  console.log(record);
}, (error) => {
  console.error(error);
});
```
