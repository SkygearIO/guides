---
title: Queries
---

[[toc]]


## Basic Queries

Skygear provides query of records with conditions. Here is a straight-forward
example of getting the note that has the `title` as `First note` inside the
public database. If you don't know the difference between
private database and public database, please read the
[Cloud Database Basics][doc-cloud-db-basics] section first.

``` javascript
const Note = skygear.Record.extend('note');
const query = new skygear.Query(Note);
query.equalTo('title', 'First note');
skygear.publicDB.query(query).then((notes) => {
  // notes is an array of Note records that has its "title" equals "First note"
}, (error) => {
  console.error(error);
});
```

You can make queries on [reserved columns][doc-reserved-columns] as well,
and of course you can put multiple conditions in the same query object:

``` javascript
const query = new skygear.Query(Note);
query.lessThan('order', 10);
query.equalTo('category', 'diary');
skygear.publicDB.query(query);
```

By default, the condition added to the query are combined with `AND`. To
construct an `OR` query, we need to specify it with the following syntax.

``` javascript
const likeQuery = new skygear.Query(Note);
likeQuery.greaterThan('like', 50);
const shareQuery = new skygear.Query(Note);
shareQuery.greaterThan('share', 10);
const query = skygear.Query.or(likeQuery, shareQuery);
query.equalTo('category', 'diary');
skygear.publicDB.query(query);
// SQL equivalent: (like > 50 OR share > 10) AND category = 'diary'
```

You can use `skygear.Query.not(query)` to get a condition negated version of `query`.


## Query conditions

Conditions are like `WHERE` clause in SQL query and `filter` in NoSQL query.
You can certainly chain conditions together. Here is a list of simple
conditions you can apply on a query:

``` javascript
const query = new skygear.Query(Note)
  .equalTo('title', 'First note')
  .notEqualTo('category', 'dangerous')
  .lessThan('age', 10)
  .lessThanOrEqualTo('age', 10)
  .greaterThan('size', 50)
  .greaterThanOrEqualTo('size', 50);
```

### Contains condition

The `contains` condition can be used to query a key for value that matches one
of the items in a specified array. Suppose here we are querying for movies,
and each movie has exactly one genre.

``` javascript
query.contains('genre', ['science-fiction', 'adventure']);
query.notContains('genre', ['romance', 'horror']);
```

If the value is an array, the `containsValue` condition can be used to query for
a key that has an array as its value containing the specified value. Suppose
further that each movie has multiple tags as an array, for example
_Independence Day: Resurgence_ may have tags "extraterrestrial" and "war".

``` javascript
query.containsValue('tags', 'war');
query.notContainsValue('tags', 'hollywood');
// both matches our example movie
```

### Like condition

The `like` function can be used to query a key for complete or partial matches
of a specified string. The percent character (`%`) can be used in place
for any number of characters while the underscore (`_`) can be used in place
for a single character.

``` javascript
query.like('genre', 'science%'); // included: science, science-fiction, etc.
query.notLike('state', 'C_');    // excluded: CA, CO, CT, etc.
query.caseInsensitiveLike('month', '%a%');    // included: January, April, etc.
query.caseInsensitiveNotLike('actor', '%k%'); // excluded: Katy, Jack, etc.
```

### Having relation condition

The `havingRelation` condition can be used to query records whose owner or
creator has relationship with the current user. For example, you can use it
to query new posts made by users whom you are following.

``` javascript
query.havingRelation('_owner', skygear.relation.Following);
query.notHavingRelation('_owner', skygear.relation.Friend);
// records owned by following users but not friend users
```

See [Social Network][doc-social-network] section for more information.

### Geolocation condition

Please visit [Data Types: Location][doc-data-type-location] section for more.


## Pagination and Ordering

### Order of records

You can sort the records based on certain field in ascending or descending order.
You can also sort on multiple fields as well.

``` javascript
query.addAscending('age');    // sorted by age increasing order
query.addDescending('price'); // sorted by price decreasing order
```

The above query will first sort from small age to large age, and records
with the same age will be then sorted from high price to low price. Just like
this SQL statement: `ORDER BY age, price DESC`.

### Pagination of records

There are 3 settings that affect query pagination:
- `limit`: number of records per query/page (default value 50)
- `page`: which page of records to return (skip `page - 1` pages of records)
- `offset`: number of records to skip

You shall not set both `page` and `offset`, otherwise `page` will be ignored.

``` javascript
query.page = 3;
/* 101st to 150th records */
```

``` javascript
query.limit = 20;
query.offset = 140;
/* 141st to 160th records */
```

``` javascript
query.limit = 15;
query.page = 8;
/* 106th to 120th records */
```

## Counting records

To get the number of records matching a query, set the `overallCount`
of the Query to `true`.

``` javascript
query.overallCount = true;
skygear.publicDB.query(query).then((notes) => {
  console.log('%d records matching query.', notes.overallCount);
}, (error) => {
  console.error(error);
});
```

The count is not affected by the limit set on the query. So, if you only want
to get the count without fetching any records, simply set `query.limit = 0`.

## Querying Skygear reserved columns

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
query.equalTo('_owner', skygear.auth.currentUser.id);
// '_owner' is an alias for '_owner_id'
skygear.publicDB.query(query);
```

Check the server [database schema][doc-database-schema] page for more.



[doc-cloud-db-basics]: /guides/cloud-db/basics/js/
[doc-reserved-columns]: /guides/cloud-db/basics/js/#reserved-columns
[doc-social-network]: /guides/social-network/basics/js/
[doc-data-type-location]: /guides/cloud-db/data-types/js/#location
[doc-data-type-reference]: /guides/cloud-db/data-types/js/#reference
