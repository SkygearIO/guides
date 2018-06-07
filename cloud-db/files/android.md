---
title:  File Storage (Assets)
---
[[toc]]

## About assets

You can make use of `Asset` to store file references such as images and videos on the database. An asset should be uploaded before being referenced by records.
Skygear automatically uploads the files to a server that you specify, like Amazon S3.

## Creating an asset

You can create an asset and upload by:

```java
byte[] data = /* Get the image data */
Asset asset = new Asset("profile.jpg", "image/jpeg", data);

this.skygear.getPublicDatabase().uploadAsset(asset, new AssetPostRequest.ResponseHandler() {
    @Override
    public void onPostSuccess(Asset asset, String response) {
        Log.i("Skygear Asset", "Successfully uploaded to " + asset.getUrl());
    }

    @Override
    public void onPostFail(Asset asset, Error error) {
        Log.i("Skygear Asset", "Upload fail: " + error.toString());
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
    public void onPartiallySaveSuccess(Map<String, Record> successRecords, Map<String, Error> errors) {
         Log.i("Skygear Record", "Partially saved");
    }

    @Override
    public void onSaveFail(Error error) {
        Log.i("Skygear Record", "Record save fails");
    }
});
```

## Frequently Asked Questions

### Is there limit to file upload on Skygear Cloud?

Currently, It is possible to upload file to Skygear Cloud with a maximum size of
50MB.

### Is it possible to add CORS settings to file upload?

If you use Skygear Cloud, a CORS settings of `*` (any origin) is added to all
files, so you can fetch your assets using scripts from any origin. If you have
your own S3 bucket, follow the [Cross-Origin Resource Sharing guide][s3-cors].
when downloading files 


[s3-cors]: https://docs.aws.amazon.com/AmazonS3/latest/dev/cors.html
