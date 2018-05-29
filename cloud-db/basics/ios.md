---
title: Cloud Database Basics
---

[[toc]]

## Introduction

Skygear allows users to store records to Skygear's PostgreSQL cloud database. You can even define your own type of `Record` and add customized fields to suit you needs.

Before you proceed, please make sure:

- You have your Skygear container configured. Simply follows the guide at [Setup Skygear][doc-setup-skygear] if you haven't.

- You have signed up as a User and logged in, as you have to be authenticated to perform any database operations. For guide on sign up and log in, go to [User Authentication Basics][doc-user-auth].

To visualize your database with graphical presentation, you can go to the Database section in [Skygear Portal][skygear-portal] and open our web data browser.


## Record
`SKYRecord` is the data storage unit in Skygear.

- `SKYRecord` must have a type.
- Each `SKYRecord ` object is like a dictionary with keys and values; keys will be mapped to database column names, and values will be stored appropriately
based on the data type.
- `SKYRecord` will be owned by the currently logged in user.
- `SKYRecord` object has a unique `id` (a string combination of record type and uuid is used).
- Each `SKYRecord` has a `recordType`, which describes the _type_ of data this record holds.
- `SKYRecord` has reserved keys that cannot be used, such as `ownerUserRecordID ` and `recordType `. Please refer to [Reserved columns][doc-reserved-columns] section for more.
- Please note Skygear database uses PostgreSQL. You are given the direct access of the database and therefore can open the database of your app using a Postgre client. Details can be found on the [Skygear Portal][skygear-portal].

A record can store whatever values that are JSON-serializable. Possible values include
strings, numbers, booleans, dates, and several other custom types that Skygear
supports.

## Database

`SKYDatabase` is the central hub of data storage in `SKYKit`. The main responsibility of database is to store `SKYRecord`s.

You will be provided with a private and a public database.

- Everything in the private database is truly private, regardless of what access
control entity you set to the record. In other words, each user has his own
private database, and only himself have access to it.
- Record saved at public database is by default public. Only the owner of the record can modify the record. Even without logging in, records in the public database can be queried (but not updated).
To control the access, you may set different access control entity to the record. However, only logged in user can do write operation on databases
- The database objects can be accessed with `[[SKYContainer defaultContainer] publicCloudDatabase]` and `[[SKYContainer defaultContainer] privateCloudDatabase]`.

Head to [Record-based ACL][doc-record-acl] and [Field-based ACL][doc-field-acl] to read more about it.

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

You will learn how to works with these data type in [Relational Records][doc-relational-record], [File storage][doc-files] and [Location, Auto-increment sequence fields][doc-data-type].

## Reserved columns

For each record type stored in the database, a table with the same name as the record type is created. For example, if your record type is called `note`, there is a table called `note` in the database. Each row in the table corresponds to one record.

For each record table there exists two types of columns, those that are reserved by Skygear and those that are user-defined. Reserved columns contain metadata of a record, such as record ID, record owner and creation time. Names of reserved columns are prefixed with underscore (`_`).

It is possible to manipulate data in record tables directly. However, one should exercise cautions when modifying data directly in record tables.

Each record table contains the following reserved columns:

| Column Name   | Object Attribute           | Description                                     |
|---------------|----------------------------|-------------------------------------------------|
| `_created_at` | `creationDate`             | `NSDate` object of when record was created      |
| `_updated_at` | `modificationDate`         | `NSDate` object of when record was updated      |
| `_created_by` | `creatorUserRecordID`      | `NSString` object of user id of record creator  |
| `_updated_by` | `lastModifiedUserRecordID` | `NSString` object of user id of record updater  |
| `_owner`      | `ownerUserRecordID`        | `NSString` object of user id of owner           |
| `_id`         | `recordID`                 | `SKYRecordID` object of record id               |

Learn how to work with the reserved columns in [More About Queries][doc-queries].


[doc-setup-skygear]: /guides/intro/quickstart/ios/
[doc-user-auth]: /guides/auth/basics/ios/
[skygear-portal]: https://portal.skygear.io
[doc-reserved-columns]: #reserved-columns
[doc-database-schema]: /guides/advanced/database-schema/
[doc-queries]: /guides/cloud-db/queries/ios/#getting-the-reserved-columns
[doc-relational-record]:/guides/cloud-db/relational-records/ios/
[doc-files]:/guides/cloud-db/files/ios/
[doc-data-type]: /guides/cloud-db/data-types/ios/
[doc-record-acl]: /guides/acl/record-acl/ios/
[doc-field-acl]: /guides/acl/field-acl/
