---
title: Data Types
---

Skygear supports almost all of the builtin JavaScript types, such as:
- String
- Number
- Boolean
- Array
- Object
- Date

There are also four other types provided by Skygear JS SDK:
- Reference
- Sequence
- Location
- Asset

<a name="reference"></a>
## Record Relations

### What Skygear provides

Skygear supports parent-child relation between records via _reference_.
`skygear.Reference` is a pointer class, which will translate to foreign key in
skygear server database for efficient query.

You can even reference a user from a record. To learn more about user object or
how to retrieve user objects, you can read the
[User Authentication Basics][doc-auth-basics].
Notice that we are not using the `new` keyword creating reference. Assume you
have the user record object `rick`.

``` javascript
const note = new Note({
  heading: 'Working Draft',
  content: 'People involved please fill in',
});
const author = new skygear.Reference(rick);
note.author = author;
skygear.publicDB.save(note);
```

You can build up reference between records.

``` javascript
const note1 = new Note({
  heading: 'Specification',
  content: 'This is first section',
});
const note2 = new Note({
  heading: 'Specification page 2',
  content: 'This is second section',
});
note1.nextPage = new skygear.Reference(note2);
skygear.publicDB.save([note2, note1]);
```

- Ordering of objects in batch save array matters if there are reference
relationship between records. In the above case, if you perform batch save on
`[note1, note2]`, there will be an error saving `note1`, but `note2` will
still be saved!
- If you wish to retrieve `note1` and `note2` at the same time in one query,
you might use `transientInclude`. Read the [Queries][doc-relational-queries]
section to learn more about eager loading.

### Deleting Referenced Record

Yet to be implemented. For now, you have to delete the referencing record
first and then the referenced record.

## Auto-Incrementing Sequence Fields

Skygear reserves the `id` field in the top level of all record as a primary key.
`id` must be unique and default to be Version 4 UUID. If you want to
auto-incrementing id for display purpose, Skygear provide a sequence for this 
purpose. The sequence is guaranteed unique. Once a record with sequence id is
saved, all the other records will automatically have sequence ids as well.

``` javascript
const note = new Note({
  content: 'Hello World'
});
note.noteID = new skygear.Sequence();

skygear.publicDB.save(note).then((note) => {
  console.log(note.noteID);
}, (error) => {
  console.log(error);
});
```

- You can omit the `noteID` on update, the value will remain unchanged.
- All the other `Note` in the database will now automatically have their
  `noteID` as well.
- You can migrate any integer to auto-incrementing sequence.
- Our JIT schema at development will migrate the DB schema to sequence. All
  `noteID` at `Note` will be a sequence type once migrated.

If you wish to override sequence manually, you can do that as well. If the
provided `noteID` is taken by another record, there will be an error; otherwise,
the record will be saved and the maximum `noteID` plus one will be used for the
next record.

``` javascript
const note = new Note({
  content: 'Hello World'
});
note.noteID = 43;
skygear.publicDB.save(note);
```

<a id="location"></a>
## Location

### Saving a location on record

``` javascript
const Photo = skygear.Record.extend('photo');
const photo = new Photo({
  'subject': 'Hong Kong',
  'location': new skygear.Geolocation(22.39649, 114.1952103),
});
```

### Querying records by distance

Get all photos taken within 400 meters of some location.

``` javascript
const reference = new skygear.Geolocation(22.283, 114.15);

const photoQuery = new skygear.Query(Photo);
photoQuery.distanceLessThan('location', reference, 400);

skygear.publicDB.query(photoQuery).then((photos) => {
  console.log(photos);
}, (error) => {
  console.error(error);
});
```

Geolocation query has `distanceGreaterThan` function as well.


### Sorting records by distance

You can sort the result of the query by distance from a reference location:

``` javascript
photoQuery.addAscendingByDistance('location', reference);
photoQuery.addDescendingByDistance('location', reference);
```

### Querying distance from record geolocation column to given reference point

You can utilize transient fields. Please read more about transient and eager
loading in the [Queries][doc-relational-queries] section.

``` javascript
photoQuery.transientIncludeDistance('location', 'distance', reference);
// 'location' is the key where Geolocation object is stored
// 'distance' is the key where calculated distance will be stored in $transient
// reference is the Geolocation object based on which distance is calculated
skygear.publicDB.query(photoQuery).then((photos) => {
  photos.forEach((photo) => {
    console.log(photo.$transient['distance']);
  });
}, (error) => {
  console.error(error);
});
```

## File Storage (Assets)

You can make use of `Asset` to store file references such as
images and videos on the database.
An asset can only be saved with a record but not as a standalone upload.
Skygear automatically uploads the files to a server that you specify,
like Amazon S3.

For example, you want to allow users to upload an image as an `attachment` to his `note`.

First you need to make a file upload form:

``` html
<input type="file" id="picture" accept="image/*">
```

Once the user has selected the image to upload, you can save it by:

``` javascript
const Note = skygear.Record.extend('note');

const picture = new skygear.Asset({
  name: '<your-asset-name>',
  file: document.getElementById('picture').files[0],
});
const note = new Note({ attachment: picture });
skygear.publicDB.save(note) // automatically upload the picture
.then((record) => {
  console.log(record.attachment.url); // where you can load the image
  // if configured properly, the url should look like the following
  // <ASSET_STORE_URL_PREFIX>/<asset-id>-<your-asset-name>
}, (error) => {
  console.error(error)
})
```

1. Call the `Asset` method to create an `Asset` object and it is stored in an asset column of the `note` object.
2. When you save the `note` object, the image is automatically uploaded to the server.

[doc-auth-basics]: /guides/auth/basics/js/
[doc-relational-queries]: /guides/cloud-db/queries/js/
