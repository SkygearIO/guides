---
title: Cloud Database Basics
---

[[toc]]


Please make sure you know about and have already configured your skygear
container before you proceed.

You can follow the steps in [Setup Skygear][doc-setup-skygear] to set it up.


## The Record Class

[Record][api-record] is the data storage unit in Skygear.

- `Record` must have a type.
- Each `Record` object contains a `java.util.Map`; keys will be mapped to database column names, and values will be stored appropriately
based on the data type. Please refer to [Data Type][doc-data-type] for more information.
- `Record` will be owned by the currently logged in user.
- `Record` object has a unique `id` (a string combination of record type and uuid is used).
- Each `Record` has a `recordType`, which describes the _type_ of data this record holds.
- `Record` has reserved keys that cannot be used, such as `ownerUserRecordID ` and `recordType `. Please refer to [Reserved Columns][doc-reserved-columns] section for more.

## Record Database

Before any record operations, you need to understand the record databases in
Skygear. You will provide with private and public database:

- Everything in private database is truly private, regardless of what access
control entity you set to the record.
- Record saved at public database is default public. To control the access, you
may set difference access control entity to the record.

In the SDK, you can get the public or private database using:

```java
// get Skygear Container
Container skygear = Container.defaultContainer(this);

// get public database
Database publicDatabase = skygear.getPublicDatabase();

// get private database
Database privateDatabase = skygear.getPrivateDatabase();
```


## Creating a record

Records in Skygear are specified via __type__ and __id__. You must provide a record
type when you create a record while the record id will be generated when you
create a record.

```java
Record aNote = new Record("Note");
```

You can assign some attributes to the record. All attribute keys should be a
string while attribute values can be in many types. Currently, all primitive
types, array of primitive types and Java `Date` type are supported.

```java
// assign some attributes
aNote.set("title", "Hello World");
aNote.set("readCount", 4);
aNote.set("lastReadAt", new Date());

// get some attributes
String title = (String) aNote.get("title");
int title = (int) aNote.get("readCount");
```

You can save a record to either __public__ or __private__ database:

```java
RecordSaveResponseHandler handler = new RecordSaveResponseHandler(){
    @Override
    public void onSaveSuccess(Record[] records) {
        Log.i(
            "Skygear Record Save",
            "Successfully saved " + records.length + " records"
        );
    }

    @Override
    public void onPartiallySaveSuccess(
        Map<String, Record> successRecords,
        Map<String, String> reasons
    ) {
        Log.i(
            "Skygear Record Save",
            "Successfully saved " + successRecords.size() + " records"
        );
        Log.i(
            "Skygear Record Save",
            reasons.size() + " records are fail to save"
        );
    }

    @Override
    public void onSaveFail(String reason) {
        Log.i(
            "Skygear Record Save",
            "Fail to save: " + reason
        );
    }
};

database.save(aNote, handler);
```

## Saving multiple records

Also, you can save multiple records at one time:

```java
Record[] records = new Record[]{ note1, note2, note3 };
database.save(records, handler);
```

You can also save multiple records atomically, which make sure the save
operation either succeeds or fails as a whole: 

```java
Record[] records = new Record[]{ note1, note2, note3 };
database.saveAtomically(records, handler);
```

## Reading a record

You can use `Query` object to find records with conditions:

```java
Query query = new Query("Note")
        .greaterThan("readCount", 3)
        .caseInsensitiveLike("title", "%hello%");
```

The following code snippet shows how to perform record query with `Query` object:

```java
RecordQueryResponseHandler handler = new RecordQueryResponseHandler() {
    @Override
    public void onQuerySuccess(Record[] records) {
        Log.i(
            "Skygear Record Query",
            String.format("Successfully got %d records", records.length)
        );
    }

    @Override
    public void onQueryError(String reason) {
        Log.i(
            "Skygear Record Query",
            "Fail to delete: " + reason
        );
    }
}

database.query(query, handler);

```

For more information, please check out the [Query Section][doc-queries]

## Updating a record

After saving the records, some meta attributes are available on the records:

```java
// the time when the record created
Date createdAt = aNote.getCreatedAt();

// the time when the record last updated
Date updatedAt = aNote.getUpdatedAt();

// the creator ID, the updater ID and the record owner ID
String creatorId = aNote.getCreatorId();
String updaterId = aNote.getUpdaterId();
String ownerId = aNote.getOwnerId();
```

## Deleting a record

You can delete a record on either __public__ or __private__ database:

```java
RecordDeleteResponseHandler handler = new RecordDeleteResponseHandler() {
    @Override
    public void onDeleteSuccess(String[] ids) {
        Log.i(
            "Skygear Record Delete",
            "Successfully deleted " + ids.length + " records"
        );
    }

    @Override
    public void onDeletePartialSuccess(String[] ids, Map<String, String> reasons) {
        Log.i(
            "Skygear Record Delete",
            "Successfully deleted " + ids.length + " records"
        );
        Log.i(
            "Skygear Record Delete",
            reasons.size() + " records are fail to delete"
        );
    }

    @Override
    public void onDeleteFail(String reason) {
        Log.i(
            "Skygear Record Delete",
            "Fail to delete: " + reason
        );
    }
});

database.delete(aNote, handler);
```

Of cause, you can delete multiple records at one time:

```java
Record[] records = new Record[]{ note1, note2, note3 };
database.delete(records, handler);
```



### Reserved Columns

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


[doc-reserved-columns]: #reserved-columns
[doc-data-type]: /guides/cloud-db/data-types/android/
[doc-queries]: /guides/cloud-db/queries/android/
[api-record]: https://docs.skygear.io/android/reference/latest/io/skygear/skygear/Record.html
