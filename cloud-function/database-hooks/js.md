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

```javascript
// Reject empty 'name' before saveing a cat to the database
skygearCloud.beforeSave('cat', function(record, original, pool, options) {
    if (record["name"] === undefined || record["name"] === null || record["name"] === '') {
    	return new Promise((resolve, reject) => {
    		reject(new Error('Missing cat name'));
    	});
    }
    return;
}, {
    async: false
});
```

In this example, the cloud code function `validate_cat_name` is registered as
a database hook by using the function `skygear.before_save`
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

## before_save hook

```javascript
 beforeSave(recordType: string, func: function(record: lib/record.js~Record, originalRecord: lib/record.js~Record, pool: pool, options: *): *, async: Boolean)
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


```javascript
skygearCloud.beforeSave('comment', function(record, original, pool, options) {
    // write your code
}, {
    async: false
});
```


- **`record` (Skygear `Record` class)**

  It is the record that is going to be saved to the database.
  The `record` is an instance of the Skygear `Record` class.
  Reading and Altering its values are done similar to how you
  manipulating a JavaScript object:

  ```javascript
  // read a value
  num = record.number

  // alter a value
  record.number = 100
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

  ```javascript
  record_type = record['_recordType'];
  record_id = record['_id'];
  owner_id = record['ownerID'];
  acl = record['_access'];

  updated_at = record['updatedAt'];
  updated_by = record['updatedBy'];
  created_at = record['createdAt'];
  created_by = record['createdBy'];
  ```


- **`original_record` (Skygear `Record` class)**

  It is the existing record object in the database, as identified by the `_id`.
  It is useful when you need to compare to the existing value when you update
  a record.

  If you are creating a new record, `original_record` will be `undefined`.


- **`pool` (Database connection pool of the Skygear PostgreSQL)**

  It is an instance of the [node-postgres](https://github.com/brianc/node-postgres/).
  
  ::: tips

  **Tips:**

  1. The database schema name is `app_<your_app_name>`, i.e. if your Skygear
     endpoint is `todo.skygeario.com`, your schema name is `app_todo`.
     Alternatively You can find your schema name by connecting to your database.

  :::

  ::: caution

  **Caution:** Any queries made using `pool` connect to the database directly.
  These queries do not pass through Skygear. Therefore they are
  not subject to [ACL][doc-acl] restrictions; and the metadata attributes
  of a record, e.g. `_updated_at`, do not get updated upon an `UPDATE` SQL
  query.

  :::

### Return Value

A record (`object`) should be returned. The returned record will be saved to the
database, instead of the `record` provided as the argument.

The record will be saved as is if you return either of the following:

- not returning at all
- `null`
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

```javascript
skygearCloud.beforeSave('selfie', function(record, original, pool, options) {
	// Check for non-empty image URL
	if (record['image_url'] === undefined || record['image_url'] === null || record['image_url'] === '') {
		 return new Promise((resolve, reject) => {
    		reject(new Error('Empty Selfie URL'));
    	});
	}

	// Set initial "like" count if it's a new record
	if (original === undefined || original === null) {
		console.log('Original record is null');
		record['likes_count'] = 0;
	}
	pool.query(`UPDATE app_hello_world.user SET last_seen = CURRENT_TIMESTAMP WHERE _id ='${record.ownerID}'`, function(err, result) {
		if (err !== undefined && err !== null) {
			console.log(err);
			return;
		}
		console.log(result);
	});

	return record;

}, {
    async: false
});
```


## after_save hook

```javascript
afterSave(recordType: string, func: function(record: lib/record.js~Record, originalRecord: lib/record.js~Record, pool: pool, options: *): *, async: Boolean)
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


```javascript
skygearCloud.afterSave('comment', function(record, original, pool, options) {
    // write your code
}, {
    async: false
});
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

```javascript
skygearCloud.afterSave('selfie', function(record, original, pool, options) {
	if (original === undefined || original === null) {
		pool.query(`UPDATE app_hello_world.user SET selfie_count = selfie_count + 1, _updated_at = CURRENT_TIMESTAMP WHERE '_id' = '${record.createdBy}'`, function(err, result) {
			if (err !== undefined && err !== null) {
				console.log(err);
				return;
			}
			console.log(result);
		})

	}
    return;
}, {
    async: false
});
```

## before_delete hook

```javascript
beforeDelete(recordType: string, func: function(record: lib/record.js~Record, originalRecord: lib/record.js~Record, pool: pool, options: *): *, async: Boolean)
```

Functions decorated with `before_delete` are executed just before a record
(of the specified record type) is deleted from the database.

Typical usages of the `before_delete` hook includes permission checks for
business logic.

### Hook function parameters

When a function is decorated with `before_delete`, it takes two parameters
when invoked, without the `original_record` parameter from the `before_save`
or `after_save` hooks:

```javascript
skygearCloud.beforeDelete('comment', function(record, original, pool, options) {
    // write your code
}, {
    async: false
});
```

These parameters are the same as
[those in a function with the `before_save` hook][before-save-func-record].
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

```javascript
// Todo
```

## after_delete hook

```javascript
afterDelete(recordType: string, func: function(record: lib/record.js~Record, originalRecord: lib/record.js~Record, pool: pool, options: *): *, async: Boolean)
```

Functions decorated with `after_delete` are executed after a record
(of the specified record type) has been deleted from the database.

Typical usages of the `after_delete` hook includes
sending notifications or database cleanups.

### Hook function parameters

When a function is decorated with `after_delete`, 
it takes two parameters like the `before_delete` hook, `record` and `db`.

```javascript
skygearCloud.afterDelete('comment', function(record, original, pool, options) {
    // write your code
}, {
    async: false
});
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

```javascript
// Todo
```

[doc-acl]: /guides/cloud-db/record-acl/ios/
[before-save-metadata]: #before-save-metadata
[before-save-func-record]: #before-save-func-record
[cloud-function-container]: /guides/cloud-function/calling-skygear-api/js/#skygear-container
