---
title: Cloud Database Basics
---

<a name="overview"></a>
## Overview

Please make sure you know about and have already configured your skygear
[container](/js/guide#set-up-app) before you proceed.

<a name="record"></a>
### The Record Class

- `Record` must have a type.
- Each `Record` object is like a dictionary with keys and values; keys will be
mapped to database column names, and values will be stored appropriately
based on the data type. Please refer to [Data Type](#data-type)
section within this guide for more information.
- `Record` will be owned by the currently logged in user.
- Each `Record` object has its unique `id` (combination of record type
  and uuid used in the database as `_id`).
- `Record` has reserved keys that cannot be used, such as `id` and `_id`.
Please refer to [Reserved Columns](#reserved) section for more.

You can design different `Record` type to model your app. Just like defining
tables in SQL.

``` javascript
const Note = skygear.Record.extend('note');
const Blog = skygear.Record.extend('blog');

const note = new Note({ 'content': 'Hello World' });
```

### Record Database

You will be provided with a private and a public database.

- Everything in the private database is truly private, regardless of what access
control entity you set to the record. In other words, each user has his own
private database, and only himself has access to it.
- Record saved at public database is by default public. Even without
logging in, records in the public database can be queried (but not updated).
To control the access, you may set different access control to the record.
- The database objects can be accessed with `skygear.publicDB` and
`skygear.privateDB`.

<a name="basic-crud"></a>
## Basic CRUD

### Creating a record

You can save a public record to server as the following.

``` javascript
skygear.publicDB.save(new Note({
  'content': 'Hello World!'
})).then((record) => {
  console.log(record);
}, (error) => {
  console.error(error);
});
```

#### Batch save

You can also batch save multiple records at one time.

``` javascript
skygear.publicDb.save([goodNote1, goodNote2, badNote3, goodNote4, badNote5])
.then((result) => {
  console.log(result.savedRecords);
  // [goodNote1, goodNote2, undefined, goodNote4, undefined]
  console.log(result.errors);
  // [undefined, undefined, error3, undefined, error5]
}, (error) => { /* request error */ })
```

By default, "good" records are still saved while "bad" records are not.
Use the `atomic` option if you don't want partial saves, so either all or none
of the records will be saved.

``` javascript
skygear.publicDB.save([goodNote, badNote], { atomic: true });
// neither of the notes are saved
```

### Reading a record

You can construct a Query object by providing a Record Type.
You can config the query by mutating its state.
Read the section about [Query](/js/guide/query) to learn more.

``` javascript
const query = new skygear.Query(Blog);
query.greaterThan('popular', 10);
query.addDescending('popular');
query.limit = 10;

skygear.publicDB.query(query).then((records) => {
  console.log(records)
}, (error) => {
  console.error(error);
})
```

### Updating a record

``` javascript
const query = new skygear.Query(Note);
query.equalTo('_id', '<your-note-_id>');

skygear.publicDB.query(query)
.then((records) => {
  const note = records[0];
  note['content'] = 'Hello New World';
  return skygear.publicDB.save(note);
}).then((record) => {
  console.log('update success');
}, (error) => {
  console.error(error);
});
```

- After saving a record, any attributes modified from the server side will
be updated on the saved record object in place.
- The local transient fields of the records are merged with any remote
transient fields applied on the server side.
- There is a shorter way for updating records, but only use it when you
know what you are doing. (Unspecified field or column will not be changed,
so in the case below only content field will be changed)

``` javascript
skygear.publicDB.save(new Note({
  _id: 'note/<your-note-_id>',
  content: 'Hello New World'
}));
```

### Deleting a record

``` javascript
skygear.publicDB.delete({
  id: 'note/<your-note-_id>'
}).then((record) => {
  console.log(record);
}, (error) => {
  console.error(error);
});
```

You can also delete multiple records at one time.

``` javascript
const query = new skygear.Query(Note);
query.lessThan('rating', 3);

const foundNotes = [];
skygear.publicDB.query(query)
.then((notes) => {
  console.log(`Found ${notes.length} notes, going to delete them.`);
  foundNotes = notes;
  return skygear.publicDB.delete(notes); // return a Promise object
})
.then((errors) => {
  errors.forEach((perError, idx) => {
    if (perError) {
      console.error('Fail to delete', foundNotes[idx]);
    }
  });
}, (reqError) => {
  console.error('Request error', reqError);
});
```

<a name="reserved"></a>
### Reserved Columns

There are quite a few reserved columns for storing records into the database.
The column names are written as **snake_case** while the JS object attributes
are mapped with **camelCase**. Please notice this one-to-one mapping. When you want
to query on reserved columns, make sure to use **snake_case**; when you get records
back as a JS object, make sure to access attributes with **camelCase**. When
creating and saving records, please avoid using attribute that is the same
as any one of the camelCase attribute names listed below.

Column Name | Object Attribute | Description
--- | --- | ---
`_created_at` | `createdAt` | date object of when record is created
`_updated_at` | `updatedAt` | date object of when record is updated last time
`_created_by` | `createdBy` | user id of record creator
`_updated_by` | `updatedBy` | user id of last record updater
`_owner_id` | `ownerID` | user id of owner
**N/A** | `id` | record type and record id
`_id` | `_id` | record id

One quick example:

``` javascript
skygear.publicDB.query(new skygear.Query(Note))
  .then((records) => console.log(records[0]));
```

``` javascript
/* Type: RecordCls */ {
  createdAt: new Date("Thu Jul 07 2016 12:12:42 GMT+0800 (CST)"),
  updatedAt: new Date("Thu Jul 07 2016 12:42:17 GMT+0800 (CST)"),
  createdBy: "118e0217-ffda-49b4-8564-c6c9573259bb",
  updatedBy: "118e0217-ffda-49b4-8564-c6c9573259bb",
  ownerID: "118e0217-ffda-49b4-8564-c6c9573259bb",
  id: "note/3b9f8f98-f993-4e1d-81c3-a451e483306b",
  _id: "3b9f8f98-f993-4e1d-81c3-a451e483306b",
  recordType: "note",
}
```

Query on reserved columns example:

``` javascript
let query = new skygear.Query(Note);
query.equalTo('_owner', skygear.currentUser.id);
// '_owner' is an alias for '_owner_id'
skygear.publicDB.query(query);
```

Check the server [database schema](/server/guide/database-schema) page for more.
