---
title: Basic CRUD
---

[[toc]]

## Creating records

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

### Creating a record

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

### Creating multiple records
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

## Querying records

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

For more information, please check out the [More About Queries][doc-queries] Section.

## Deleting records

### Deleting a record

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

### Deleting multiple records

Similar to the above handlers, you can delete multiple records at a time.

```java
Record[] records = new Record[]{ note1, note2, note3 };
publicDatabase.delete(records, handler);
```
```kotlin
val records = arrayOf(note1, note2, note3)
publicDatabase.delete(records, handler)
```


[doc-queries]: /guides/cloud-db/queries/android/
