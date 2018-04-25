---
title: Basic CRUD
---

[[toc]]

Before working with the Skygear records, we suggest you reading the [basics](/guides/cloud-db/basics/js/).

## Creating records

### Creating a record
You can save a public record to server as the following:

``` javascript
skygear.publicDB.save(new Note({
  'content': 'Hello World!'
})).then((record) => {
  console.log(record);
}, (error) => {
  console.error(error);
});
```
Here we created a new record in 'Note' and under the 'content' column, we added the record value: 'Hello World!'. Your data browser should look similar to the following, but with the 'note' record type instead of the 'test' record type.
![Web database viewer](/assets/common/quickstart-database-viewer.png)

### Creating multiple records

You can also batch save multiple records at one time.

``` javascript
skygear.publicDB.save([goodNote1, goodNote2, badNote3, goodNote4, badNote5])
.then((result) => {
  console.log(result.savedRecords);
  // [goodNote1, goodNote2, undefined, goodNote4, undefined]
  console.log(result.errors);
  // [undefined, undefined, error3, undefined, error5]
}, (error) => { /* request error */ })
```

### Atomic

By default, "good" records are still saved while "bad" records are not.
Use the `atomic` option if you don't want partial saves, so either all or none
of the records will be saved.

``` javascript
skygear.publicDB.save([goodNote, badNote], { atomic: true });
// neither of the notes are saved
```

## Updating records

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

## Querying records

You can construct a Query object by providing a Record Type.
You can config the query by mutating its state.
Read the section about [More About Queries][doc-queries] to learn more.

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

## Deleting records

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
### Deleting multiple records
You can also delete multiple records at one time.

``` javascript
const query = new skygear.Query(Note);
query.lessThan('rating', 3);

let foundNotes = [];
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

[doc-setup-skygear]: /guides/get-started/js/
[doc-data-type]: /guides/cloud-db/data-types/js/
[doc-reserved-columns]: #reserved-columns
[doc-database-schema]: /guides/advanced/database-schema/
[doc-queries]: /guides/cloud-db/queries/js/
