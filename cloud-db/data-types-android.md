---
title: Data Types
---

<a name="reference"></a>
## Record Relations

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

<a name="auto-increment"></a>
## Auto-Incrementing Sequence Fields

## Location

The location data type represents a point on Earth using
its latitude and longitude. It is saved in the PostgreSQL database using
the geographic object type provided by the
[PostGIS](http://postgis.net/) extension.

### Saving a location in a record

You can save a location into a record attribute by providing a
[Location object](https://developer.android.com/reference/android/location/Location.html)
as the second argument of the `set` method on a record.

Correspondingly, the `get` method for a location data type on a record
gives you a `Location` object so that you can obtain its latitude and longitude.

```java
Container skygear = Container.defaultContainer(this);
Database publicDB = skygear.getPublicDatabase();

Location office = new Location("gps");
office.setLatitude(22.336265);
office.setLongitude(114.147932);

Record aRecord = new Record("Meeting");
aRecord.set("place", office);

publicDB.save(aRecord, new RecordSaveResponseHandler(){
    @Override
    public void onSaveSuccess(Record[] records) {
        Record savedRecord = records[0];
        Location place = savedRecord.get("place");
        Log.i(
            "Skygear Location Demo",
            String.format(
                "The place is at (%f, %f)",
                place.getLatitude(),
                place.getLongitude()
            )
        );
    }

    @Override
    public void onPartiallySaveSuccess(
        Map<String, Record> successRecords,
        Map<String, String> reasons
    ) {
        // Partial success handling
    }

    @Override
    public void onSaveFail(String reason) {
        // Error handling
    }
});
```

### Query by distance from a specified location

You can perform queries on the location data type using its distance from
a specified location with `distanceLessThan` or `distanceGreaterThan`.
Also, you can sort the query result by the distance with
`addAscendingByDistance` or `addDescendingByDistance`.

For example, if you have a database table `Restaurant` containing a
list of restaurants with its location defined in the `location` column,
you can obtain a list of restaurants within 100m from the office,
sorted from the nearest to the farthest:

```java
Container skygear = Container.defaultContainer(this);
Database publicDB = skygear.getPublicDatabase();

Location office = new Location("gps");
office.setLatitude(22.336265);
office.setLongitude(114.147932);

Query restaurantQuery = new Query("Restaurant")
        .distanceLessThan("location", office, 100)
        .addAscendingByDistance("location", office);

publicDB.query(restaurantQuery, new RecordQueryResponseHandler() {
    // Implementation of response handler
});
```

### Obtaining the distance from a specified location in a query

The above example fetches the restaurants within 100m from the office
but the actual distances to each restaurant are unknown.
To have the distance calculated, you can use `transientIncludeDistance`
on the query object.

The method `transientIncludeDistance` takes three arguments:

```
transientIncludeDistance(key, mapToKey, loc)
```

- `key` (string): the attribute (field) name on the Record type to be queried.
  The field should be of `location` type.
- `mapToKey` (string): the key that will be used to access the computed
  distance from `getTransient()`
- `loc` (`Location` object): the location from which the distance will be
  calculated

```java
Container skygear = Container.defaultContainer(this);
Database publicDB = skygear.getPublicDatabase();

Location office = new Location("gps");
office.setLatitude(22.336265);
office.setLongitude(114.147932);

// Bank.location is a location field
// the distance can be accessed via 'distanceFromOffice'
Query bankQuery = new Query("Bank")
        .transientIncludeDistance("location", "distanceFromOffice", office);

publicDB.query(bankQuery, new RecordQueryResponseHandler() {
    @Override
    public void onQuerySuccess(Record[] records) {
        for (int idx = 0; idx < records.length; idx++) {
            Record perRecord = records[idx];
            double distance = (double) perRecord.getTransient().get("distance");

            Log.i("Skygear", "Distance: " + distance);
        }
    }

    @Override
    public void onQueryError(String reason) {
        // Error handling
    }
});
```

## File Storage (Assets)

### Uploading and associating an asset to a record

You can make use of `Asset` to store file references such as images and videos on the database. An asset should be uploaded before being referenced by records.
Skygear automatically uploads the files to a server that you specify, like Amazon S3.

You can create an asset and upload by:

```java
byte[] data = /* Get the image data */
Asset asset = new Asset("profile.jpg", "image/jpeg", data);

this.skygear.uploadAsset(asset, new AssetPostRequest.ResponseHandler() {
    @Override
    public void onPostSuccess(Asset asset, String response) {
        Log.i("Skygear Asset", "Successfully uploaded to " + asset.getUrl());
    }

    @Override
    public void onPostFail(Asset asset, String reason) {
        Log.i("Skygear Asset", "Upload fail: " + reason);
    }
});
```

Each asset should have a name. However, asset names will never collide.
That means you can upload multiple assets with the same asset name.

Asset name (i.e. `asset.getName()`) is rewritten after the asset being uploaded.

Please be reminded that the asset URL (i.e. `asset.getUrl()`) will be populated
with an expiry URL after fetching / querying the record from server.

After uploading the asset, you can set it as a field of an record.

```java
Record aNote = new Record("Note");
aNote.set("attachment", asset);

this.skygear.getPublicDatabase().save(aNote, new RecordSaveResponseHandler(){
    @Override
    public void onSaveSuccess(Record[] records) {
        Log.i("Skygear Record", "Successfully saved");
    }

    @Override
    public void onSaveFail(String reason) {
        Log.i("Skygear Record", "Record save fails");
    }
});
```
