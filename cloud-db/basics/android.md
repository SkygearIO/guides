---
title: Cloud Database Basics
---

[[toc]]


Please make sure you have already configured your skygear
container before you proceed.

You can follow the steps in [Setup Skygear][doc-setup-skygear] to set it up.


## The Record Class

[Record][api-record] is the data storage unit in Skygear.

- Each `Record` must be given a `type`. Such `type` is simply a `String` property of
`Record` describing the type of data this record holds.
- Each `Record` object contains a `java.util.Map` representing its attributes; Keys will be mapped to database column names, and values will be stored accordingly based
on the data type. Please refer to [Data Type][doc-data-type] for more information.
- There exist some reserved keys that cannot be used as a key for an attribute. Examples are `ownerUserRecordID ` and `recordType `. Please refer to [Reserved Columns][doc-reserved-columns] section for more information.
- Each `Record` created will be owned by the currently logged in user.
- Each `Record` object has a unique `id` (a string combination of record type and uuid is used).


## Record Database

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


## Creating a Record

In Skygear, you can specify a __type__ and an __id__ for a `Record`. A record type must be given when a record is instantiated, while its __id__ will be generated afterwards if not given. We will talk more
about `Record` __id__ with examples in a later section.

```java
Record note1 = new Record("Note");
```
```kotlin
val note1 = Record("Note")
```

You can assign attributes to a `Record`. A key of an attribute must be a `String` while its value can be of various types. All primitive types, array of primitive types and Java `Date` type are currently supported.

Below are some examples on how attributes can be set and retrieved.

```java
// assign some attributes
note1.set("title", "Hello World");
note1.set("readCount", 4);
note1.set("lastReadAt", new Date());

// get some attributes
String title = (String) note1.get("title");
int readCount = (int) note1.get("readCount");
Date lastReadAt = (Date) note1.get("lastReadAt");
```
```kotlin
// assign some attributes
note1.set("title", "Hello World")
note1.set("readCount", 4)
note1.set("lastReadAt", Date())

// get some attributes
val title = note1.get("title") as String
val readCount = note1.get("readCount") as Int
val lastReadAt = note1.get("lastReadAt") as Date
```

## Saving Records

To save just a single record or multiple records, a `RecordSaveResponseHandler` has to be instantiated first.
If a record has been given an ID that already exists on the cloud
database, saving this record will update the existing record.

```java
RecordSaveResponseHandler handler = new RecordSaveResponseHandler(){

  @Override
  public void onSaveSuccess(Record[] records) {
      for (int i = 0; i < records.length; i++) {
          Record record = records[i];
          String recordId = record.getId();
          String recordTitle = (String) record.get("title");
          Object content = record.getData();
          Log.i("Skygear Record Save", "Successfully saved:" +
                  "\n- Record ID: " + recordId +
                  "\n- Record Title: " + recordTitle +
                  "\n- Data: " + content
          );
      }
  }

  @Override
  public void onPartiallySaveSuccess(Map<String, Record> successRecords, Map<String, Error> errors) {
      for (Map.Entry<String, Record> entry : successRecords.entrySet()) {
          String recordId = entry.getKey();
          Record record = entry.getValue();
          String recordTitle = (String) record.get("title");
          Object content = record.getData();
          Log.i("Skygear Record Save", "Successfully saved:" +
                  "\n- Record ID: " + recordId +
                  "\n- Record Title: " + recordTitle +
                  "\n- Data: " + content
          );
      }

      for (Map.Entry<String, Error> entry : errors.entrySet()) {
          String recordId = entry.getKey();
          String errorMessage = entry.getValue().getMessage();
          Log.e("Skygear Record Save", "Failed to save record with id: " + recordId + " - " + errorMessage);
      }
  }

  @Override
  public void onSaveFail(Error error) {
      Log.e("Skygear Record Save",  "Failed to save - " + error.getMessage());
  }

};
```
```kotlin
val handler = object : RecordSaveResponseHandler() {

    override fun onSaveSuccess(records: Array<Record>) {
        records.forEach { record ->
            val recordId = record.id
            val recordTitle = record.get("title") as String
            val content = record.data
            Log.i(
                    "Skygear Record Save",
                    """
                    Successfully saved:
                    - Record ID: $recordId
                    - Record Title: $recordTitle
                    - Data: $content
                    """
            )
        }
    }

    override fun onPartiallySaveSuccess(successRecords: Map<String, Record>, errors: Map<String, Error>) {
        successRecords.forEach { (recordId, record) ->
            val recordTitle = record.get("title") as String
            val content = record.data
            Log.i(
                    "Skygear Record Save",
                    """
                    Successfully saved:
                    - Record ID: $recordId
                    - Record Title: $recordTitle
                    - Data: $content
                    """
            )
        }

        errors.forEach { (recordId, error) ->
            Log.e("Skygear Record Save", "Failed to save record with id: $recordId - ${error.message}")
        }
    }

    override fun onSaveFail(error: Error) {
        Log.e("Skygear Record Save", "Failed to save - ${error.message}")
    }

}
```

A record can be saved to either the __public__ or __private__ database. Here is an example saving the record `note1` we have created above to a public database.

```java
Database publicDatabase = skygear.getPublicDatabase();
publicDatabase.save(note1, handler);
```
```kotlin
val publicDatabase = skygear.publicDatabase
publicDatabase.save(note1, handler)
```

To save multiple records at one time, put them in an array:

```java
Record[] records = new Record[]{ note1, note2, note3 };
publicDatabase.save(records, handler);
```
```kotlin
val records = arrayOf(note1, note2, note3)
publicDatabase.save(records, handler)
```

You can also save multiple records atomically, where result of the save operation can either be Success or Fail. Note that the callback `onPartiallySaveSuccess` will not be triggered in such case.

```java
Record[] records = new Record[]{ note1, note2, note3 };
publicDatabase.saveAtomically(records, handler);
```
```kotlin
val records = arrayOf(note1, note2, note3)
publicDatabase.saveAtomically(records, handler)
```

## Reading a record

Record(s) can be queried with conditions. You will have to
specify the conditions in a `Query` first.

```java
Query query = new Query("Note")
        .greaterThan("readCount", 3)
        .caseInsensitiveLike("title", "%hello%");
```
```kotlin
val query = Query("Note")
        .greaterThan("readCount", 3)
        .caseInsensitiveLike("title", "%hello%")
```

The following code snippet shows how to perform record query with `Query` object:

```java
RecordQueryResponseHandler handler = new RecordQueryResponseHandler() {
    @Override
    public void onQuerySuccess(Record[] records) {
        for (int i = 0; i < records.length; i++) {
            Record record = records[i];
            String recordId = record.getId();
            String recordTitle = (String) record.get("title");
            Object content = record.getData();
            Log.i("Skygear Record Query", "Successfully queried:" +
                    "\n- Record ID: " + recordId +
                    "\n- Record Title: " + recordTitle +
                    "\n- Data: " + content
            );
        }
    }

    @Override
    public void onQueryError(Error error) {
        Log.e(
                "Skygear Record Query",
                "Failed to query - " + error.getMessage()
        );
    }
};

publicDatabase.query(query, handler);
```
```kotlin
val handler = object : RecordQueryResponseHandler() {
    override fun onQuerySuccess(records: Array<Record>) {
        records.forEach { record ->
            val recordId = record.id
            val recordTitle = record.get("title") as String
            val content = record.data
            Log.i(
                    "Skygear Record Query",
                    """
                    Successfully queried:
                    - Record ID: $recordId
                    - Record Title: $recordTitle
                    - Data: $content
                    """
            )
        }
    }

    override fun onQueryError(error: Error) {
        Log.e(
                "Skygear Record Query",
                "Failed to query - ${error.message}"
        )
    }
}

publicDatabase.query(query, handler)
```

For more information, please check out the [Query Section][doc-queries].

## Metadata of Record

After a record has been saved, some metadata will be generated automatically. You can get the metadata as follows:

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

## Deleting a record

You can delete a record on either the __public__ or __private__ database.
Here, we are going to use the record `note1` that was created and saved
above to demonstrate the process of deleting a record.

```java
RecordDeleteResponseHandler handler = new RecordDeleteResponseHandler() {

    @Override
    public void onDeleteSuccess(String[] ids) {
        for (int i = 0; i < ids.length; i++) {
            Log.i(
                    "Skygear Record Delete",
                    "Successfully deleted record with ID: " + ids[i]
            );
        }
    }

    @Override
    public void onDeletePartialSuccess(String[] ids, Map<String, Error> errors) {
        for (int i = 0; i < ids.length; i++) {
            Log.i(
                    "Skygear Record Delete",
                    "Successfully deleted record with ID: " + ids[i]
            );
        }

        for (Map.Entry<String, Error> entry : errors.entrySet()) {
            String recordId = entry.getKey();
            String errorMessage = entry.getValue().getMessage();
            Log.e("Skygear Record Delete", "Failed to delete record with id: " + recordId + " - " + errorMessage);
        }
    }

    @Override
    public void onDeleteFail(Error error) {
        Log.e("Skygear Record Delete", "Failed to delete - " + error.getMessage());
    }

};

publicDatabase.delete(note1, handler);
```
```kotlin
val handler = object : RecordDeleteResponseHandler() {

    override fun onDeleteSuccess(ids: Array<String>) {
        ids.forEach { id ->
            Log.i(
                    "Skygear Record Delete",
                    "Successfully deleted record with ID: " + id
            )
        }
    }

    override fun onDeletePartialSuccess(ids: Array<String>, errors: Map<String, Error>) {
        ids.forEach { id ->
            Log.i(
                    "Skygear Record Delete",
                    "Successfully deleted record with ID: " + id
            )
        }

        errors.forEach { (id, error) ->
            Log.e("Skygear Record Delete", "Failed to delete rmessageecord with id: $id - ${error.message}")
        }
    }

    override fun onDeleteFail(error: Error) {
        Log.e("Skygear Record Delete", "Failed to delete - ${error.message}")
    }

}

publicDatabase.delete(note1, handler)
```

Similar to the above handlers, you can delete multiple records at a time.

```java
Record[] records = new Record[]{ note1, note2, note3 };
publicDatabase.delete(records, handler);
```
```kotlin
val records = arrayOf(note1, note2, note3)
publicDatabase.delete(records, handler)
```

### Reserved Columns

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


[doc-reserved-columns]: #reserved-columns
[doc-data-type]: /guides/cloud-db/data-types/android/
[doc-queries]: /guides/cloud-db/queries/android/
[api-record]: https://docs.skygear.io/android/reference/latest/io/skygear/skygear/Record.html
