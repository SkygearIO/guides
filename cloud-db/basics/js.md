---
title: Cloud Database Basics
---

[[toc]]

Please make sure you know about and have already configured your skygear
container before you proceed.

You can follow the steps in [Setup Skygear][doc-setup-skygear] to set it up.

## The Skygear record

- `Record` must have a type.
- Each `Record` object is like a dictionary with keys and values; keys will be
mapped to database column names, and values will be stored appropriately
based on the data type. Please refer to [Data Type][doc-data-type]
section for more information.
- `Record` will be owned by the currently logged in user.
- Each `Record` object has its unique `id` (combination of record type
  and uuid used in the database as `_id`).
- `Record` has reserved keys that cannot be used, such as `id` and `_id`.
Please refer to [Reserved Columns][doc-reserved-columns] section for more.
- Please note Skygear database uses PostgreSQL. You can review our [tips](https://docs.skygear.io/guides/intro/quickstart/js/#tips-anchor) on the 3 ways you can access the Skygear database.

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

## Public database vs private database

You will be provided with a private and a public database.

- Everything in the private database is truly private, regardless of what access
control entity you set to the record. In other words, each user has his own
private database, and only himself has access to it.
- Record saved at public database is by default public. Even without
logging in, records in the public database can be queried (but not updated).
To control the access, you may set different access control to the record.
- The database objects can be accessed with `skygear.publicDB` and
`skygear.privateDB`.

## Supported data type

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

You will learn how to works with these data type in [Working with relational records][doc-relational-record], [File storage](doc-files) and [Working with other data types][doc-data-type].

## Skygear reserved columns

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

Learn how to query the Skygear reserved columns in [More About Queries](/guides/cloud-db/queries/js/#querying-skygear-reserved-columns).

[doc-setup-skygear]: /guides/get-started/js/
[doc-data-type]: /guides/cloud-db/data-types/js/
[doc-reserved-columns]: #reserved-columns
[doc-database-schema]: /guides/advanced/database-schema/
[doc-queries]: /guides/cloud-db/queries/js/
[doc-relational-record]:/guides/cloud-db/relational-records/js/
[doc-files]:/guides/cloud-db/files/js/
