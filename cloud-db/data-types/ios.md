---
title: Location, Auto-increment Sequence Fields
---

[[toc]]

## Location

1. Supported by saving `CLLocation` in `SKYRecord`
2. Only latitude and longitude of `CLLocation` is recorded

### Saving a location on record

```obj-c
#import <CoreLocation/CoreLocation.h>

CLLocation *location = [[CLLocation alloc] initWithLatitude:22.283 longitude:114.15];
record[@"location"] = location;
```

```swift
import CoreLocation

let location = CLLocation(latitude: 22.283, longitude: 114.15)
record.setObject(location, forKey: "location" as NSCopying!)
```

See
[Location and Maps Programming Guide][doc-apple-location-and-maps-programming-guide]
for help on getting current location.

### Querying records by distance

Get all photos taken within 400 meters of some location.

```obj-c
CLLocation *distanceFromLoc = [[CLLocation alloc] initWithLatitude:22.283 longitude:114.15];
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"distanceToLocation:fromLocation:(location, %@) < %f", distanceFromLoc, 400.f];
SKYQuery *query = [SKYQuery queryWithRecordType:@"photo" predicate:predicate];

[[[SKYContainer defaultContainer] privateCloudDatabase] performQuery:query completionHandler:^(NSArray *photos, NSError *error) {
    if (error) {
        NSLog(@"error querying photos: %@", error);
        return;
    }

    NSLog(@"got %@ photos taken within 400m of (22.283, 114.15)", @(photos.count));
}];
```

```swift
let distanceFromLoc = CLLocation(latitude: 22.283, longitude: 114.15)
let predicate = NSPredicate(format: "distanceToLocation:fromLocation:(location, %@) < %f", distanceFromLoc, 400.0)
let query = SKYQuery(recordType: "photo", predicate: predicate)

SKYContainer.default().privateCloudDatabase.perform(query) { (photos, error) in
    if error != nil {
        print ("error querying photos: \(error)")
        return
    }

    if let photos = photos {
        print ("got \(photos.count) photos taken within 400m of (22.283, 114.15")
    }
}
```

`distanceToLocation:fromLocation:` is a Skygear function which returns distance
between a location field in a record and a specific location.

### Sorting records by distance

Cont. from the example above.

```obj-c
query.sortDescriptors = @[[SKYLocationSortDescriptor locationSortDescriptorWithKey:@"location"
                                                                 relativeLocation:distanceToLoc
                                                                        ascending:YES]];
```

```swift
query.sortDescriptors = [SKYLocationSortDescriptor.init(key: "location",
                                                         relativeLocation: distanceFromLoc,
                                                         ascending: true)]
```

### Retrieving record location field distances relative to a point

Utilize transient fields.

```obj-c
NSExpression *distanceFunction = [NSExpression expressionWithFormat:@"distanceToLocation:fromLocation:(location, %@)", distanceToLoc];
query.transientIncludes = @{@"distance": distanceFunction};
```

```swift
let distanceFunction = NSExpression(format: "distanceToLocation:fromLocation:(location, %@)", distanceFromLoc)
query.transientIncludes = ["distance": distanceFunction]
```

Then we can access the distance in `completionHandler` like this:

```obj-c
[[[SKYContainer defaultContainer] privateCloudDatabase] performQuery:query completionHandler:^(NSArray *photos, NSError *error) {
    if (error) {
        NSLog(@"error querying photos: %@", error);
        return;
    }

    for (SKYRecord *photo in photos) {
        NSNumber *distance = photo.transient[@"distance"];
        NSLog(@"Photo taken from distance = %@", distance);
    }
}];
```

```swift
SKYContainer.default().privateCloudDatabase.perform(query) { (photos, error) in
    if error != nil {
        print ("error query photos: \(error)")
        return
    }

    for photo in photos as! [SKYRecord] {
        let distance = photo.transient.object(forKey: "distance")
        print ("Photo taken from distance = \(distance)")
    }
}
```

## Auto-increment sequence fields
### Make use of sequence object

Skygear reserves the `id` field in the top level of all record as a primary key.
`id` must be unique and default to be Version 4 UUID. If you want to
auto-incrementing id for display purpose, Skygear provide a sequence for this
purpose. The sequence is guaranteed unique.

```obj-c
SKYRecord *todo = [SKYRecord recordWithRecordType:@"todo"];
todo[@"title"] = @"Write documents for Skygear";
todo[@"noteID"] = [SKYSequence sequence];

SKYDatabase *privateDB = [[SKYContainer defaultContainer] privateCloudDatabase];
[privateDB saveRecord:todo completion:^(SKYRecord *record, NSError *error) {
    if (error) {
        NSLog(@"error saving todo: %@", error);
        return;
    }

    NSLog(@"saved todo with auto increment noteID = %@", record[@"noteID"]);
}];
```

```swift
let todo = SKYRecord(recordType: "todo")
todo.setObject("Write documents for Skygear", forKey: "title" as NSCopying!)
todo.setObject(SKYSequence(), forKey: "noteID" as NSCopying!)

let privateDB = SKYContainer.default().privateCloudDatabase
privateDB.save(todo, completion: { (record, error) in
    if error != nil {
        print ("error saving todo: \(error)")
        return
    }

    print ("saved todo with auto increment noteID = \(record?.object(forKey: "noteID"))")
})
```

- You can omit the `noteID` on update, the value will remain unchanged.
- All the other `Note` in the database will now automatically have their
  `noteID` as well.
- You can migrate any integer to auto-incrementing sequence.
- Our JIT schema at development will migrate the DB schema to sequence. All
  `noteID` at `Note` will be a sequence type once migrated.

### Override sequence manually

```obj-c
SKYRecord *todo = [SKYRecord recordWithRecordType:@"todo"];
todo[@"title"] = @"Override noteID";
todo[@"noteID"] = @43;

SKYDatabase *privateDB = [[SKYContainer defaultContainer] privateCloudDatabase];
[privateDB saveRecord:todo completion:^(SKYRecord *record, NSError *error) {
    if (error) {
        NSLog(@"error saving todo: %@", error);
        return;
    }

    NSLog(@"saved todo with noteID == 43, %@", record[@"noteID"]);
}];
```
```swift
let todo = SKYRecord(recordType: "todo")
todo.setObject("Write documents for Skygear", forKey: "title" as NSCopying!)
todo.setObject(43, forKey: "noteID" as NSCopying!)

let privateDB = SKYContainer.default().privateCloudDatabase
privateDB.save(todo, completion: { (record, error) in
    if error != nil {
        print ("error saving todo: \(error)")
        return
    }

    print ("saved todo with noteID == 43, \(record?.object(forKey: "noteID"))")
})
```


[doc-reference]: #reference
[doc-sequence]: #sequence
[doc-location]: #location
[doc-assets]: #assets
[doc-apple-location-and-maps-programming-guide]: https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/LocationAwarenessPG/CoreLocation/CoreLocation.html
