---
title: More About Queries
---

[[toc]]

Skygear provides query of records with conditions. You can apply condition to
Skygear queries, only getting the records you want.


## Basic Queries

To perform a query, first you need to construct a `Query` object:

```java
Query noteQuery = new Query("Note")
        .equalTo("title", "Hello world")
        .greaterThan("rating", 3);
        .addAscending("rating");
```

After constructing a `Query` object, you can perform a query as following:

```java
Container skygear = Container.defaultContainer(this);
Database publicDB = skygear.getPublicDatabase();

publicDB.query(noteQuery, new RecordQueryResponseHandler() {
    @Override
    public void onQuerySuccess(Record[] records) {
        Log.i("Record Query", String.format("Successfully got %d records", records.length));
    }

    @Override
    public void onQueryError(String reason) {
        Log.i("Record Query", String.format("Fail with reason:%s", reason));
    }
});
```

### Or operation

`Or` operation can be applied to multiple `Query` objects. The usage is as
following:

```java
Query query1 = new Query("Note").caseInsensitiveLike("title", "%important%");
Query query2 = new Query("Note").greaterThan("rating", 3);
Query query3 = new Query("Note").greaterThan("readCount", 10);

Query compoundQuery = Query.or(query1, query2, query3)
compoundQuery.addAscending("rating");

// perform query using `compoundQuery` object...
```

### Negate operation

`negate` operation can be applied to a `Query` object to negate the whole query
predicate.

```java
// (rating > 1) AND (rating < 3)
Query betweenOneAndThree = new Query("Note")
        .greaterThan("rating", 1)
        .lessThan("rating", 3);

// NOT ((rating > 1) AND (rating < 3))
Query notBetweenOneAndThree = new Query("Note")
        .greaterThan("rating", 1)
        .lessThan("rating", 3)
        .negate();

```


## Query conditions

Besides the operations shown above, the following list out all operations supported.

- `like`
- `notLike`
- `caseInsensitiveLike`
- `caseInsensitiveNotLike`
- `equalTo`
- `notEqualTo`
- `greaterThan`
- `greaterThanOrEqualTo`
- `lessThan`
- `lessThanOrEqualTo`
- `contains`
- `notContains`
- `containsValue`
- `notContainsValue`
- `negate`
- `addDescending`
- `addAscending`


## Pagination and Ordering

Refer to [Query][api-query] class reference for pagination and order in query
result.

## Getting the reserved columns
For each new record type stored in the database, a table with the same name as the record type is created. For example, if your record type is called `Note` which no record of this type has been saved before, a new table called `Note`
will be created. Each row in the table corresponds to one record of type `Note`.

For each record table there exists two types of columns (fields of a record), those reserved by Skygear and those user-defined. Reserved columns contain metadata of a record, such as record ID, record owner and creation time. These reserved columns are prefixed with underscore, like `_created_at`.

It is possible to manipulate data in record tables directly. However, one should exercise with caution when modifying data directly in record tables.

Each record table contains the following reserved columns:

| Column Name   | Object Attribute           | Description                                     |
|---------------|----------------------------|-------------------------------------------------|
| `_created_at` | `creationDate`             | `NSDate` object of when record was created      |
| `_updated_at` | `modificationDate`         | `NSDate` object of when record was updated      |
| `_created_by` | `creatorUserRecordID`      | `NSString` object of user id of record creator  |
| `_updated_by` | `lastModifiedUserRecordID` | `NSString` object of user id of record updater  |
| `_owner`      | `ownerUserRecordID`        | `NSString` object of user id of owner           |
| `_id`         | `recordID`                 | `SKYRecordID` object of record id               |


You can retrieve the values from the object by accessing its properties:

```java
// the time when the record created
Date createdAt = note1.getCreatedAt();

// the time when the record last updated
Date updatedAt = note1.getUpdatedAt();

// the creator ID, the updater ID and the record owner ID
String creatorId = note1.getCreatorId();
String updaterId = note1.getUpdaterId();
String ownerId = note1.getOwnerId();
```
```kotlin
// the time when the record created
val createdAt = note1.createdAt

// the time when the record last updated
val updatedAt = note1.updatedAt

// the creator ID, the updater ID and the record owner ID
val creatorId = note1.creatorId
val updaterId = note1.updaterId
val ownerId = note1.ownerId
```
Please head to [Database Schema][doc-database-schema] to read more about Reserved Columns, Record Tables and Reserved Tables.

[doc-database-schema]:/guides/advanced/database-schema/
[api-query]:https://docs.skygear.io/android/reference/latest/io/skygear/skygear/Query.html
