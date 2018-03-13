---
title: User Relations Basics
---

[[toc]]
:::caution
This is an experimental feature. Official support not available.
:::

## Introduction

Skygear provide three default relations: friend, following and follower.
- `follower` and `following` are one non-mutual relation with direction. You can
follow a user without his explicit approval.
- `friend` is mutual relation without direction. If you add Ben as your friend
and Ben add you as his friend, then you two will have `friend` relationship.


## Adding Relations

If the current user wants to follow Ben:

``` javascript
const toFollow = new skygear.Following([ben]);
// ben is a user object
skygear.relation.add(toFollow).then((result) => {
  console.log(result);
}, (error) => {
  console.error(error);
});
```

The `result` variable you get in the callback function is a `RelationResult`
instance that have the following keys/attributes:
- `result.success` is an array of user objects with whom relations are added
- `result.fail` is an array of user objects with whom relations fail to be added
- `result.partialError` is a boolean that equals true when some relations are
  added while others fail to be added

How can you get the user object? You can either search by email or simply
construct a new user object with the correct user id. Read the section
about [Users][doc-search-users] to learn more about searching by email.

``` javascript
const ben = new skygear.User({
  user_id: "<ben's-user-id>",
});
```

If the current user wants to add friend with Ben:

``` javascript
const beFriend = new skygear.Friend([ben]);
skygear.relation.add(beFriend).then(...);
```


## Querying Relations

You may query the relations of the current user or other users.

To get Ben's follower list:

``` javascript
skygear.relation.queryFollower(ben).then((users) => {
  console.log(users);
}, (error) => {
  console.error(error);
});
```

To get the current user's friend list:

``` javascript
skygear.relation.queryFriend(skygear.auth.currentUser).then(...);
```

To get the current user's following list:

``` javascript
skygear.relation.queryFollowing(skygear.auth.currentUser).then(...);
```

To get the 101-200th friends of the current user:

``` javascript
skygear.relation.queryFriend(skygear.auth.currentUser, {
  page: 2, limit: 100,
}).then((users) => {
  console.log(users);
}, (error) => {
  console.error(error);
});
```

Alternatively you can use the query syntax if you want more conditions.
Learn more in the [Queries][doc-queries] section.

``` javascript
const query = new skygear.relation.Query(skygear.relation.Friend);
query.user = skygear.auth.currentUser;
query.limit = 10;
query.page = 2;
query.greaterThan('age', 20);
skygear.relation.query(query).then((users) => {
  console.log(users.overallCount);
  // The total count matching the relation regardless of page or limit
}, (error) => {
  console.error(error);
});
```


## Removing Relations

``` javascript
const unFollow = new skygear.relation.Follower([ben]);
skygear.relation.remove(unFollow).then((result) => {
  console.log(result.success);
  // an array with one element: user ID of ben
}, (error) => {
  console.error(error);
});
```

[doc-search-users]: /guides/auth/user-profile/js/#search-users
[doc-queries]: /guides/cloud-db/queries/js/
