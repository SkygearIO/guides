---
title: Database Hooks
---

[[toc]]

## Overview

Database hooks are executed when a database record is modified through Skygear.
It allows you to run custom codes before or after
a record is created, updated or deleted.
This is useful when you have to perform operations like data validation and
sending emails based on the change in database records,

```python
# Reject empty 'name' before saving a cat to the database
@skygear.before_save('cat', async=False)
def validate_cat_name(record, original_record, db):
    if not record.get('name'):
        raise Exception('Missing cat name')
    return record
```

In this example, the cloud code function `validate_cat_name` is registered as
a database hook by using the function decorator `skygear.before_save`
and specifying the record type `cat` (i.e. database
table name) that it should hook to.

You can have multiple functions registered with the same database hook.
They will all be executed but the execution order is not guaranteed.
If this is not desired, you can call each of the functions from
a single database hook.

## Available Database Hooks

There are 4 types of database hooks, registered with the function
decorators listed below.

These function decorators are available from the `skygear` module, i.e.
you can use `skygear.before_save` after `import skygear`.

- `before_save(record_type, async=True)`: to run before a record is created or updated
- `after_save(record_type, async=True)`: to run after a record is created or updated
- `before_delete(record_type, async=True)`: to run before a record is deleted
- `after_delete(record_type, async=True)`: to run after a record is deleted

Decorator parameters:

- `record_type` (string): the record type (table name) the function
  should hook to
- `async` (boolean): whether the function should be executed asynchronously
  (Default is `True`). If it is set to `False`, the client SDK will only
  receive the response of a database operation only after the database hook
  has finished its execution.

::: note

In Python 3.7, `async` is a reserved word. If you are using py-skygear 1.6.0
 or later, you can use `async_` instead of `async`. Example:

```python
@skygear.before_save('cat', async_=False)
def validate_cat_name(record, original_record, db):
    pass
```

:::

## before_save hook

```python
before_save(record_type, async=True)
```

Functions decorated with `before_save` are executed just before a record
(of the specified record type) is created or updated in the database.

Typical usages of the `before_save` hook includes data validation, setting
default values and checking permissions.

::: caution

**Caution:** `async` must be set to `False` in order for the `before_save`
hook to work properly. It includes modifying record attributes or 
raising an exception to abort the save operation.

:::

### Hook function parameters

When a function is decorated with `before_save`, it takes three parameters
when invoked:

```python
@skygear.before_save('comment', async=False)
def my_func(record, original_record, db):
    # write your code
    pass
```


- **`record` (Skygear `Record` class)**

  It is the record that is going to be saved to the database.
  The `record` is an instance of the Skygear `Record` class.
  Reading and Altering its values are done similar to how you
  manipulating a [Python dictionary][python-dict]:

  ```python
  # read a value
  num = record.get('number')

  # alter a value
  record['number'] = 100
  ```

  ::: note

  **Note:** This `record` is a complete object including all attributes.
  It means that even when you are updating an existing record by providing only
  one attribute, you still have the full record object containing
  the existing values of other attributes.

  :::



  For the metadata attributes, they can be accessed but not altered.
  Among those attributes, only the record ID, record type,
  owner ID and ACL reflect the latest values as they will be saved to
  the database.
  Values of the other attributes (`updated_at`, `updated_by`,
  `created_at` and `created_by`), if existed, are the
  existing ones, e.g. the `updated_at` is the time
  the record was last updated.

  ```python
  record_type = record.id.type
  record_id = record.id.key
  owner_id = record.owner_id
  acl = record.acl

  # only to retrieve the previous values for these
  updated_at = record.updated_at
  updated_by = record.updated_by
  created_at = record.created_at
  created_by = record.created_by
  ```


- **`original_record` (Skygear `Record` class)**

  It is the existing record object in the database, as identified by the `_id`.
  It is useful when you need to compare to the existing value when you update
  a record.

  If you are creating a new record, `original_record` will be `None`.


- **`db` (SQLAlchemy connection)**

  It is an instance of the [SQLAlchemy engine connection][sqlalchemy-conn]
  that you can use to interact with the database directly. Common use cases
  include running additional database queries or updating other database records
  using `db.execute`.

  ::: tips

  **Tips:**

  1. You are advised to use the [Textual SQL][sqlalchemy-textual-sql]
     feature to bind parameters in queries.

  2. The database schema name is `app_<your_app_name>`, i.e. if your Skygear
     endpoint is `todo.skygeario.com`, your schema name is `app_todo`.
     Alternatively You can find your schema name by connecting to your database.

  :::

  ::: caution

  **Caution:** Any queries made using `db` connect to the database directly.
  These queries do not pass through Skygear. Therefore they are
  not subject to [ACL][doc-acl] restrictions; and the metadata attributes
  of a record, e.g. `_updated_at`, do not get updated upon an `UPDATE` SQL
  query.

  :::

### Return Value

A record (`dict`) should be returned. The returned record will be saved to the
database, instead of the `record` provided as the argument.

The record will be saved as is if you return either of the following:

- not returning at all
- `None`
- the `record` in the argument

If you raise an exception, the record will not be saved.
An `UnexpectedError` will be returned, with the Exception message in the
`message` attribute.

### Example

The following `before_save` hook example on the `Selfie` record type
demonstrates:

1. data validation (`image_url` cannot be empty)
2. setting a default value (`likes_count` is zero)
3. executing a database SQL query (update the user's last seen time)


```python
from datetime import datetime
import sqlalchemy as sa
import skygear
from skygear.utils.context import current_user_id


@skygear.before_save('selfie', async=False)
def before_save_selfie(record, original_record, db):

    # check for non-empty image URL
    if record.get('image_url') is None:
        raise Exception('Empty Selfie URL')

    # set initial "like" count if it's a new record
    if original_record is None:
        record['likes_count'] = 0

    # update some other table
    # e.g. update user last seen time
    sql = sa.text('''
        UPDATE "app_helloworld"."user"
        SET last_seen = :current_time
        WHERE "_id" = :user_id
    ''')
    db.execute(
        sql,
        user_id=current_user_id(),
        current_time=datetime.now(),
    )

    return record
```

## after_save hook

```python
after_save(record_type, async=True)
```

Functions decorated with `after_save` are executed after a record
(of the specified record type) is created or updated in the database.

It is different from the `before_save` hook that in an `after_save` hook
the `record` has been saved to the database. This means that we cannot
alter the `record` in place or stop the `record` from being saved by
raising an exception.

Typical usages of the `after_save` hook includes operations that take
significant time to complete, e.g. sending emails, or
updating records related to the newly saved record.

### Hook function parameters

When a function is decorated with `after_save`, it takes three parameters
when invoked:

```python
@skygear.after_save('comment', async=False)
def my_func(record, original_record, db):
    # write your code
    pass
```

These parameters are the same as
[those in a function decorated with the `before_save` hook][before-save-func-record].
The only difference is that the `record` in the `after_save` hook contains all
up-to-date metadata attributes, whereas the `before_save` hook only has
[a few][before-save-metadata] up-to-date.

::: tips

**Tips**: If you want to alter the `record`, you can either use
the provided `db` connection to execute raw SQL, or
[call the Skygear API using the container][cloud-function-container].

:::

### Return Value

No return value is necessary.

### Example

The following `after_save` hook example on the `selfie` record type
demonstrates:

- updating the user's selfie count after a selfie is successfully saved

```python
from datetime import datetime
import sqlalchemy as sa
import skygear
from skygear.utils.context import current_user_id


@skygear.after_save('selfie', async=True)
def after_save_selfie(record, original_record, db):
    # increment user selfie count if it is a new selfie record
    if original_record is None:
        sql = sa.text('''
            UPDATE "app_helloworld"."user"
            SET selfie_count = selfie_count + 1,
                _updated_at = :current_time
            WHERE "_id" = :user_id
        ''')
        db.execute(
            sql,
            user_id=current_user_id(),
            current_time=datetime.now(),
        )
```

## before_delete hook

```python
before_delete(record_type, async=True)
```

Functions decorated with `before_delete` are executed just before a record
(of the specified record type) is deleted from the database.

Typical usages of the `before_delete` hook includes permission checks for
business logic.

### Hook function parameters

When a function is decorated with `before_delete`, it takes two parameters
when invoked, without the `original_record` parameter from the `before_save`
or `after_save` hooks:

```python
@skygear.before_delete('comment', async=False)
def my_func(record, db):
    # write your code
    pass
```

These parameters are the same as
[those in a function decorated with the `before_save` hook][before-save-func-record].
This time the `record` contains all up-to-date metadata attributes.

### Return Value

No return value is necessary.

If you raise an exception, the record will not be deleted.
An `UnexpectedError` will be returned, with the Exception message in the
`message` attribute.

::: caution

**Caution:** `async` must be set to `False` if you need to cancel the
delete operation by raising an exception.

:::

### Example

The following `before_delete` hook example on the `group_chat_user` record type
demonstrates:

- a feature that the last admin in a group chat cannot be deleted

```python
import sqlalchemy as sa
import skygear


# not to allow removing the last admin user in a group
# a group chat user has the following attributes:
# - role: admin/member
# - chat: (foreign key to Chat table) the chat it belongs to
@skygear.before_delete("group_chat_user", async=False)
def before_delete_group_chat_user(record, db):
    if record['role'] == 'admin':
        chat_id = record['chat'].recordID.key
        s = sa.text('''
            SELECT COUNT(*) FROM app_chat.group_chat_user
            WHERE "chat" = :chat_id and role='admin'
        ''')
        count = db.execute(s, chat_id=chat_id).first()['count']
        if count == 1:
            raise Exception("Cannot remove last group admin")
```

## after_delete hook

```python
after_delete(record_type, async=True)
```

Functions decorated with `after_delete` are executed after a record
(of the specified record type) has been deleted from the database.

Typical usages of the `after_delete` hook includes
sending notifications or database cleanups.

### Hook function parameters

When a function is decorated with `after_delete`, 
it takes two parameters like the `before_delete` hook, `record` and `db`.

```python
@skygear.after_delete('comment', async=False)
def my_func(record, db):
    # write your code
    pass
```

These parameters are the same as
[those in a function decorated with the `before_save` hook][before-save-func-record].
This time the `record` contains all up-to-date metadata attributes.

### Return Value

No return value is necessary.

### Example

The following `after_delete` hook example on the `group_chat_user` record type
demonstrates:

- sending email to the user when he is removed from the group chat

```python
import sqlalchemy as sa
import skygear


@skygear.after_delete('group_chat_user', async=False)
def after_delete_group_chat_user(record, db):
    group_chat_id = record['group'].recordID.key
    decrement_query = sa.text('''
        UPDATE app_chat.group_chat
        SET user_count = user_count - 1
        WHERE _id = :group_id
    ''')
    db.execute(decrement_query, group_id = group_chat_id)
```

[python-dict]: https://docs.python.org/3.6/tutorial/datastructures.html#dictionaries
[sqlalchemy-conn]: http://docs.sqlalchemy.org/en/latest/core/connections.html#sqlalchemy.engine.Connectioh
[sqlalchemy-textual-sql]: http://docs.sqlalchemy.org/en/latest/core/tutorial.html#using-textual-sql
[doc-acl]: /guides/cloud-db/acl/ios/
[before-save-metadata]: #before-save-metadata
[before-save-func-record]: #before-save-func-record
[cloud-function-container]: /guides/cloud-function/calling-skygear-api/python/#skygear-container
