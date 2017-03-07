---
title: Calling Skygear API
---

<a name="skygear-container"></a>
## Skygear Container

Besides using the SDKs to interact with Skygear,
you can also call the Skygear APIs directly using the Skygear
`Container` from the cloud code.

```python
from skygear.container import SkygearContainer

container = SkygearContainer(api_key="your-api-key")
```

To use the `SkygearContainer` function, you need to import it from the
`skygear.container` module, and configure the API key.

You can use `container.send_action` to send requests to the Skygear server.

As an example, you can perform a query through Skygear with the following:

```python
from skygear.container import SkygearContainer

container = SkygearContainer(api_key="your-api-key")

# finding the `task` record with the given ID
query = {
    'database_id': '_public',
    'record_type': 'task',
    'limit': 1,
    'predicate': [
        'eq',
        {
            '$type': 'keypath',
            '$val': '_id',
        },
        "cdfc7bb4-afd3-464c-a430-c9564c2202cf",
    ],
}

response = container.send_action('record:query', query)
records = response.get('result', []) # returned rows are in 'result'
# `records` is an array of dict
task = records[0]
return task
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

```python
from skygear.options import options

master_key = options.masterkey
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

```python
import uuid
from skygear.container import SkygearContainer
from skygear.options import options # to obtain the master key

master_container = SkygearContainer(
    api_key=options.masterkey,
    user_id="1a5f91c9-b9c0-487d-8d48-bff0b156f38d"
)

# When you call master_container.send_action,
# the action will be done on behalf of the given user

task_record = {
    '_access': [{'level': 'read', 'public': True}], # public read
    '_id': 'task/' + str(uuid.uuid4()), # generate an ID
    'description': 'Complete the sales report',
    'reviewer': { # a foreign key to user
        '$type': 'ref',
        '$id': 'user/1a5f91c9-b9c0-487d-8d48-bff0b156f38d',
    },
}

master_container.send_action(
    'record:save',
    {
        'database_id': '_public',
        'records': [task_record],
    }
)
```

Deleting a record can be done in a similar fashion:

```python
from skygear.container import SkygearContainer
from skygear.options import options # to obtain the master key

master_container = SkygearContainer(
    api_key=options.masterkey,
    user_id="1a5f91c9-b9c0-487d-8d48-bff0b156f38d"
)

master_container.send_action(
    "record:delete",
    {
        "database_id": "_public",
        "ids": ["task/cdfc7bb4-afd3-464c-a430-c9564c2202cf"]
    }
)
```

<a name="database-queries"></a>
## Database Queries

In the [database hooks][doc-cloud-code-db-hooks], you receive the `db` argument
which is an instance of the [SQLAlchemy engine connection].
In other types of cloud code functions, 
you can obtain such an instance by importing `db` and call
`db.conn()` from the `skygear.utils` module.

```python
import skygear
from skygear.utils import db

@skygear.op('connect_db')
def db_conn_demo():
    with db.conn() as conn:
        # conn is an instance of SQLAlchemy engine connection
        # e.g. call conn.execute to run raw SQL
        pass
```

<a name="pubsub-events"></a>
## PubSub Events

You can publish a message to a PubSub channel through cloud code using
the `publish` function in the `skygear.pubsub` module.

```python
from skygear.pubsub import publish

publish('my_channel', {'text': 'Hello World'})
```

The `publish` function has no return values and takes two arguments:

- **channel** (String)

  The name of the PubSub channel to publish to.
  All subscribers to the channel will receive the data.

- **data** (dictionary)

  This is the data to be published to the channel.

<a name="push-notifications"></a>
## Push Notifications

You can send push notifications to users from the cloud code
using the `push_user` function in the `skygear.action` module.
Below sample code use the notification payload for GCM, which is default for Skygear Android SDK.

```python
from skygear.container import SkygearContainer
from skygear.action import push_user

container = SkygearContainer(api_key="your-api-key")
user_id = "abcd-efgh"
notification = {
    'apns': {
        'aps': {
            'alert': {
                'title': title,
                'body': message,
            }
        },
        'from': 'skygear',
        'operation': 'notification',
    },
    'gcm': {
         'notification': {
              'title': title,
              'body': message,
          }
    },
}

response = push_user(container, user_id, notification)
```

## Parameters

```python
push_user(container, user_id, notification)
```

- **container** (`SkygearContainer`)

  An instance of the `SkygearContainer`, with the API key configured.

- **user_id** (String)

  The Skygear user ID who will receive the push notification. The prefix
  `user/` is not necessary for the user ID.

- **notification** (dictionary)

  It should be a Python dictionary with two keys, `apns` and `gcm`,
  representing the argument for
  [Apple Push Notification Service][apns]
  and [Google Cloud Messaging][gcm]
  respectively.

If you need to send push notifications to multiple users,
you can pass a list of user IDs (Python list) to the `push_users` function
(don't miss the ending "s" of `push_users`!).
The other arguments remain the same.

```python
user_ids = ["abcd-efgh", "efgh-abcd", "aceg-bdfh"]

response = push_users(container, user_ids, notification)
```

### Return Value

(TODO)

[portal-app-settings]: https://portal.skygear.io/app/settings
[doc-cloud-code-db-hooks]: /guides/cloud-function/database-hooks/python/
[gcm]: https://developers.google.com/cloud-messaging/
[apns]: https://developer.apple.com/go/?id=push-notifications
