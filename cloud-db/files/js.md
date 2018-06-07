---
title: File Storage
---

[[toc]]

## About assets

You can make use of `Asset` to store file references such as
images and videos on the database.
An asset can only be saved with a record but not as a standalone upload.
Skygear automatically uploads the files to a S3 bucket assigned to your app.
You can also use your own file storage bucket. More settings can be found at the Developer Portal.

## Creating an asset

The common use case is to create an asset using the File API using
an HTML form with a file input field.

First you need to create a file upload input field:

``` html
<input type="file" id="picture" accept="image/*">
```

Once the user has selected the image to upload, you can create an Asset object by:

``` javascript
const picture = new skygear.Asset({
  name: 'your-asset-name',
  file: document.getElementById('picture').files[0],
});
```

After creating an Asset, you need to attach it to a record.
Skygear will upload the asset when the record is saved.

For example, you want to allow users to upload an image as an `attachment`
to his `note`:

```javascript
const Note = skygear.Record.extend('note');

const note = new Note({ attachment: picture });
skygear.publicDB.save(note) // automatically upload the picture
.then((record) => {
  console.log(record.attachment.url);
  // if configured properly, the url should look like the following
  // <ASSET_STORE_URL_PREFIX>/<asset-id>-<your-asset-name>
}, (error) => {
  console.error(error)
})
```

## Creating an Asset with a base64 string

Apart from creating an Asset through the File API, you can also create one using
a base64 string.

``` javascript
const picture = new skygear.Asset({
  name: 'hello.png',            // filename of your asset
  base64: 'iVBORw0KGgoAAAA...', // base64 of the file, no mime
  contentType: 'image/png'      // mime of the file
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
