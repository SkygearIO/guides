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

- `Record` must have a type.
- Each `Record` object is like a dictionary with keys and values; keys will be
mapped to database column names, and values will be stored appropriately
based on the data type.
section for more information.
- `Record` will be owned by the currently logged in user.
- Each `Record` object has its unique `id` (combination of record type
  and uuid used in the database as `_id`).
- `Record` has reserved keys that cannot be used, such as `id` and `_id`.
Please refer to [Reserved Columns][doc-reserved-columns] section for more.
- Please note Skygear database uses PostgreSQL. You are given the direct access of the database and therefore can open the database of your app using a Postgre client. Details can be found on the [Skygear Portal][skygear-portal].

You can design different `Record` types to model your app. Just like defining
tables in SQL.

``` javascript
const Note = skygear.Record.extend('note');
  //You can use 'Note' to reference the 'note' record type.

const note = new Note({ 'content': 'Hello World' });
  //This creates a new record value 'Hello World' stored under a column called 'content' in the 'note' record type.

const Blog = skygear.Record.extend('blog');
  //You can also define a different record type (e.g. a blog).


```

## Database

You will be provided with a private and a public database.

- Everything in the private database is truly private, regardless of what access
control entity you set to the record. In other words, each user has his own
private database, and only himself has access to it.
- Record saved at public database is by default public. Even without
logging in, records in the public database can be queried (but not updated).
To control the access, you may set different access control to the record.
- The database objects can be accessed with `skygear.publicDB` and
`skygear.privateDB`.

## Data type

Skygear supports most built-in JavaScript types:
- String
- Number
- Boolean
- Array
- Object
- Date

There are also four other types provided by Skygear JS SDK:
- Reference (relational records)
- Asset (files)
- Sequence
- Location

You will learn how to works with these data type in [Relational Records][doc-relational-record], [File storage][doc-files] and [Location, Auto-increment sequence fields][doc-data-type].

## Reserved columns

There are quite a few reserved columns for storing records into the database. Those contain the metadata of a record.
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

Learn how to work with the reserved columns in [More About Queries][doc-queries].

[doc-user-auth]: /guides/auth/basics/js/
[skygear-portal]: https://portal.skygear.io
[doc-setup-skygear]: /guides/intro/quickstart/js/
[doc-data-type]: /guides/cloud-db/data-types/js/
[doc-reserved-columns]: #reserved-columns
[doc-database-schema]: /guides/advanced/database-schema/
[doc-queries]: /guides/cloud-db/queries/js/#getting-the-reserved-columns
[doc-relational-record]:/guides/cloud-db/relational-records/js/
[doc-files]:/guides/cloud-db/files/js/
