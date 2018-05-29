---
title: Relational Records
---

[[toc]]

## Creating references

Skygear supports parent-child relations between records via _reference_.
`Reference` is a pointer class, which will translate to foreign key in
skygear server database for efficient queries.

You can create a reference by passing an object to the `Reference` class
constructor:

```java
Record aPost = new Record("Post", postData);
Reference aPostReference = new Reference(aPost);
```

By setting the reference to a record, you can create a relation between
the two records:

```java
Record aComment = new Record("Comment", commentData);
aComment.set("post", aPostReference);

```

## Querying referenced records

This example shows how to query all notes (`Note` record) who has an `account` field reference to a user record. In this example, we will query all notes where `account` equals to the current user.

```java
Record currentUser = Container.defaultContainer(this).getAuth().getCurrentUser(); // Get the current user
Query noteQuery = new Query("Note").equalTo("account", currentUser.getId());

Database publicDB = Container.defaultContainer(this).getPublicDatabase();

publicDB.query(noteQuery, new RecordQueryResponseHandler() {
    @Override
    public void onQuerySuccess(Record[] records) {
        Log.i("Record Query", String.format("Successfully got %d records", records.length));

        for (Record record : records){
            Log.i("Record Query", record.toJson().toString());
        }
    }

    @Override
    public void onQueryError(Error error) {
        Log.i("Record Query", String.format("Fail with reason:%s", error.getLocalizedMessage()));
    }
```

If you haven't have the corresponding record in hand (in this example, we will use the User record `182654c9-d205-43aa-8e74-d465c830087a`), you can reference with a specify `id` without making another query in this way:

```java
Query noteQuery = new Query("Note").equalTo("account", "182654c9-d205-43aa-8e74-d465c830087a");

Database publicDB = Container.defaultContainer(this).getPublicDatabase();

publicDB.query(noteQuery, new RecordQueryResponseHandler() {
    @Override
    public void onQuerySuccess(Record[] records) {
        Log.i("Record Query", String.format("Successfully got %d records", records.length));

        for (Record record : records){
            Log.i("Record Query", record.toJson().toString());
        }
    }

    @Override
    public void onQueryError(Error error) {
        Log.i("Record Query", String.format("Fail with reason:%s", error.getLocalizedMessage()));
    }

```

## Eager loading

Skygear supports eager loading of referenced records when you are querying the
referencing records. It is done by specify the transient include on the query.
The transient field will be available through the `getTransient` method.

```java
/* Assume the `Order` table has a field named `shipping_address`,
   which is a foreign key to a table named `address`
*/
Query orderQuery = new Query("Order"); /* build the query as usual */
orderQuery.transientInclude("shipping_address");

// after getting a record of the query result
Map<String, Record> transientMap = record.getTransient();
Record shippingAddress = (Record) transientMap.get("shipping_address");

```

By default you access the transient attributes by using the field name.
If you prefer another key, you can specify it by providing the second
argument to `transientInclude`:

```java
orderQuery.transientInclude("shipping_address", "shipping");
Record shippingAddress = (Record) transientMap.get("shipping");

```
