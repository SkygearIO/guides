---
title: User Profile
---

<a name="user-profile"></a>
## User Profile

Whenever a new user signs up, a user profile is automatically created for
you to track user information other than their username, email or password.

The user profile is created using the record type [`user`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/user.js~User.html) with
the column `_id` storing the user ID, so you can use it
in the same way as using any other record types.
You can query and update a user's profile by manipulating using
the [`User`](https://doc.esdoc.org/github.com/skygeario/skygear-SDK-JS/class/lib/user.js~User.html) record type.

Important note: This user profile is created in the public database, i.e.
it is visible to any other user. Therefore you should not store any sensitive
information in this record type. You will need to create another record type
with the private database for information that can only be accessed by the
owner.

### Saving to the current user's profile

``` javascript
// skygear.UserRecord is equivalent to skygear.Record.extend('user')
const modifiedProfile = new skygear.UserRecord({
  '_id': 'user/' + skygear.currentUser.id,
  'language': 'en-US',
  'gender': 'male',
  'age': 20,
});
skygear.publicDB.save(modifiedProfile).then((profile) => {
  console.log(profile); // updated user record
});
```

### Retrieving the current user's profile

``` javascript
const query = new skygear.Query(skygear.UserRecord);
query.equalTo('_id', skygear.currentUser.id);
skygear.publicDB.query(query).then((records) => {
  const profile = records[0];
  console.log(profile);
}, (error) => {
  console.error(error);
});
```

<a id="search-users"></a>
### Retrieving another user's profile by email or username

You can retrieve the public profiles of other users by using their emails or
usernames. You can provide either a single email/username or an array of
emails/usernames.
The promise will be resolved by an array of matched user profiles.

``` javascript
// you can also pass an array of emails
skygear.discoverUserByEmails('ben@skygear.com').then((users) => {
  console.log(users); // array of profiles (user records)
}, (error) => {
  console.error(error);
});
```

``` javascript
// you can also pass an array of usernames
skygear.discoverUserByUsernames('ben').then((users) => {
  console.log(users); // array of profiles (user records)
}, (error) => {
  console.error(error);
});
```
