---
title: Location, Auto-increment sequence
---

[[toc]]

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

## Auto-Increment Sequence Fields

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

[doc-auth-basics]: /guides/auth/basics/js/
[doc-relational-queries]: /guides/cloud-db/queries/js/
