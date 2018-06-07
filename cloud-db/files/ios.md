---
title: Files Storage
---

[[toc]]

## About assets

You can make use of `Asset` to store file references such as images and videos on the database. An asset can only be saved with a record but not as a standalone upload. Skygear automatically uploads the files to a S3 bucket assigned to your app. You can also use your own file storage bucket. More settings can be found at the Developer Portal.

## Creating an asset

For example, you want to allow users to upload an image as an `image` to his `SKYRecord`. Once the user has selected the image to upload, you can save it by:

```obj-c
- (void)imagePickerController:(UIImagePickerController *)picker
didFinishPickingMediaWithInfo:(NSDictionary<NSString *, id> *)info
{
    UIImage *image = info[UIImagePickerControllerOriginalImage];
    SKYAsset *asset = [SKYAsset assetWithName:@"profile-picture"
                                         data:UIImagePNGRepresentation(image)];
    asset.mimeType = @"image/png";

    SKYContainer *container = [SKYContainer defaultContainer];
    [container.privateCloudDatabase uploadAsset:asset completionHandler:^(SKYAsset *asset, NSError *error) {
        if (error) {
            NSLog(@"error uploading asset: %@", error);
            return;
        }

        self.photoRecord[@"image"] = asset;
        [container.privateCloudDatabase saveRecord:self.photoRecord completion:nil];
    }];
}
```

```swift
func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [String : Any]) {
    if let url = info[UIImagePickerControllerReferenceURL] as? URL {
        let asset = SKYAsset(name: "profile-picture", fileURL: url)
        asset.mimeType = "image/png"

        let container = SKYContainer.default()
        container.privateCloudDatabase.uploadAsset(asset, completionHandler: { (asset, error) in
            if error != nil {
                print ("error uploading asset: \(error)")
                return
            }

            self.photoRecord?.setObject(asset, forKey: "image" as NSCopying!)
            container.privateCloudDatabase.save(self.photoRecord, completion: nil)
        })
    }
}
```

Asset names will never collide. i.e. you can upload multiple assets with the same asset name.

`asset.name` is rewritten after the asset being uploaded.

```obj-c
NSString *newName = asset.name;
// The name is changed to "c36e803a-f333-47bf-a3e9-4d52b660a71a-profile-picture"
// from "profile-picture"
```

```swift
let newName = asset.name
// The name is changed to "c36e803a-f333-47bf-a3e9-4d52b660a71a-profile-picture"
// from "profile-picture"
```

## Querying an asset

`SKYAsset.url` will be populated with an expiry URL after fetching /
querying the record from server.


```obj-c
[[[SKYContainer defaultContainer] privateCloudDatabase] fetchRecordWithID:photoRecordID completionHandler:^(SKYRecord *photo, NSError *error) {
    if (error) {
        NSLog(@"error fetching photo: %@", error);
        return;
    }

    SKYAsset *asset = photo[@"image"];
    NSURL *url = asset.url;
    // do something with the url
}];
```

```swift
SKYContainer.default().privateCloudDatabase.fetchRecord(with: photoRecordID) { (photo, error) in
    if error != nil {
        print ("error fetching photo: \(error)")
        return
    }

    if let asset = photo?.object(forKey: "image") as? SKYAsset{
        let url = asset.url
        // do something with the url
    }
}
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
