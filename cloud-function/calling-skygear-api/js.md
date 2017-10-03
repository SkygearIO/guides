---
title: Calling Skygear API
---

[[toc]]


## Skygear Container

Besides using the SDKs to interact with Skygear,
you can also call the Skygear APIs directly using the Skygear
`skygearCloud.getContainer()` from the cloud code.

```javascript
const container = skygearCloud.getContainer();
```

You can create Skygear Container object by `CloudCodeContainer()`
from `skygearCloud`. And configure it with `apiKey`, `endPoint` and `userId`.

```javascript
const container = new skygearCloud.CloudCodeContainer();
container.apiKey = 'your-api-key';
container.endPoint = skygearCloud.settings.skygearEndpoint + '/';
```

Query can be done similar to JS SDK.

As an example, you can perform a query through Skygear with the following:

```javascript
const container = skygearCloud.getContainer('admin'); // act as `admin`
const TaskRecord = skygear.Record.extend('task');
const query = new skygear.Query(TaskRecord);
query.equalTo('_id', 'cdfc7bb4-afd3-464c-a430-c9564c2202cf');
container.publicDB.query(query).then((records) => {
	console.log(records);
});
```

::: caution

**Caution:** By using the app's API key, the `SkygearContainer`
does not have an authenticated user associated.
Any actions or queries made are assumed to be made by an
unauthenticated user, and are subject to access control set
to the public. Therefore you cannot save or alter records
with this API key. To have a user-aware `SkygearContainer`
for such operations, you need to use the master key, explained below.

:::

### Using the master key

A master key is a special key which allows you to perform operations
that would not be possible using the normal API key.
A typical use case in the cloud code is to impersonate a user for
creating, altering or querying records.

You can find your master key in your
[app settings in the Skygear portal][portal-app-settings],
or obtain your master key through the cloud code by:

```javascript
masterKey = skygearCloud.settings.masterKey;
```

::: caution

**Caution:** You should never expose the master key in the client SDK.

:::

You can impersonate a user from the cloud code by setting the master
key as the `api_key` when you initialize the `SkygearContainer`,
and provide the user ID as the `user_id` parameter. You do not
need an access token for the user to be impersonated.

Any calls made using `send_action` will then be done on behalf of
the provided user.

The following example demonstrates saving a record on behalf of
the authenticated user:

```javascript
const container = skygearCloud.getContainer('admin');

// When you call container to save the record,
// the action will be done on behalf of the given user
const Task = skygear.Record.extend('task');
const TaskRecord = new Task({
	description: 'Complete the sales report'});
container.publicDB.save(TaskRecord).then((result) => {
	console.log(result);
});
```

Deleting a record can be done in a similar fashion:

```javascript
const container = skygearCloud.getContainer('admin');

container.publicDB.delete({
	id: 'note/cdfc7bb4-afd3-464c-a430-c9564c2202cf'
}).then((record) => {
	console.log(record);
}, (error) => {
	console.error(error);
});
```


## Database Queries

In the [database hooks][doc-cloud-code-db-hooks], you receive the `db` argument
which is an instance of the [SQLAlchemy engine connection].
In other types of cloud code functions, 
you can obtain such an instance by importing `db` and call
`db.conn()` from the `skygear.utils` module.

(TODO)


## PubSub Events

You can publish a message to a PubSub channel through cloud code using
the `publish` function in the `skygear.pubsub` module.

```javascript
container.pubsub.publish('my_channel', {'text': 'Hello World'});
```

The `publish` function has no return values and takes two arguments:

- **channel** (String)

  The name of the PubSub channel to publish to.
  All subscribers to the channel will receive the data.

- **data** (Object)

  This is the data to be published to the channel.


## Push Notifications

Not yet implemented in JS.

### Return Value

(TODO)

[portal-app-settings]: https://portal.skygear.io/app/settings
[doc-cloud-code-db-hooks]: /guides/cloud-function/database-hooks/python/
[gcm]: https://developers.google.com/cloud-messaging/
[apns]: https://developer.apple.com/go/?id=push-notifications
