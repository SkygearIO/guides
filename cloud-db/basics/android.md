---
title: Cloud Database Basics
---

[[toc]]


## Introduction

Skygear allows users to store records to Skygear's PostgreSQL cloud database.
You can even define your own type of `Record` and add customized fields to suit
you needs.

Before you proceed, please make sure:
- You have your Skygear container configured. Simply follows the guide at
[Setup Skygear][doc-setup-skygear] if you haven't.

- You have signed up as a `User` and logged in, as you have to be authenticated
to perform any database operations. For guide on sign up and log in, go to
[User Authentication Basics][doc-user-auth].

To visualize your database with graphical presentation, you can go to the Database section in [Skygear Portal][skygear-portal]
and open our web data browser.

## Record

[Record][api-record] is the data storage unit in Skygear.

- Each `Record` must be given a `type`. Such `type` is simply a `String` property of
`Record` describing the type of data this record holds.
- Each `Record` object contains a `java.util.Map` representing its attributes; Keys will be mapped to database column names, and values will be stored accordingly based
on the data type.
- There exist some reserved keys that cannot be used as a key for an attribute. Examples are `ownerUserRecordID ` and `recordType `. Please refer to [Reserved Columns][doc-reserved-columns] section for more information.
- Each `Record` created will be owned by the currently logged in user.
- Each `Record` object has a unique `id` (a string combination of record type and uuid is used).

## Database
Before doing any record operations, you have to understand how databases
in Skygear work. You are provided with a private and a public database:

- Every record in the private database is truly private, regardless of the [Access Control List (ACL)](https://docs.skygear.io/guides/cloud-db/acl-overview/android/) set.

- Records saved at the public database is set to public by default. ACL can be set per record.

The public and private database can be retrieved using:

```java
// get Skygear Container
Container skygear = Container.defaultContainer(this);

// get public database
Database publicDatabase = skygear.getPublicDatabase();

// get private database
Database privateDatabase = skygear.getPrivateDatabase();
```
```kotlin
// get Skygear Container
val skygear = Container.defaultContainer(this)

// get public database
val publicDatabase = skygear.publicDatabase

// get private database
val privateDatabase = skygear.privateDatabase
```

## Data type
Below are the data types Skygear supports:
- String
- Number
- Boolean
- Array
- Object
- Date

There are also four other types provided by the Skygear SDK:
- Reference (relational records)
- Asset (files)
- Sequence
- Location

You will learn how to works with these data type in [Relational records][doc-relational-record], [File storage][doc-files] and [Location, Auto-increment Sequence fields][doc-data-type].

## Reserved columns

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

Learn how to get the metadata in [More About Queries][doc-queries].

[doc-relational-record]: /guides/cloud-db/relational-records/android/
[doc-files]: /guides/cloud-db/files/android/
[doc-setup-skygear]: /guides/intro/quickstart/android/
[doc-queries]: /guides/cloud-db/queries/android/#getting-the-reserved-columns
[doc-user-auth]: /guides/auth/basics/android/
[doc-reserved-columns]: #reserved-columns
[doc-data-type]: /guides/cloud-db/data-types/android/
[skygear-portal]:https://portal.skygear.io/apps
[api-record]: https://docs.skygear.io/android/reference/latest/io/skygear/skygear/Record.html
