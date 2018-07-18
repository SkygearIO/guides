---
title: Error Handling in SDK
description: How to handle errors gracefully when something goes wrong
---

[[toc]]

# Error handling

Whenever you perform an action that calls the server, there is chance the
server rejects your request for various reasons.
This guide demonstrate how should the errors be handled.

For example when you try to save a record to the database and without
logging in, you get an error.

```obj-c
SKYDatabase *publicDB = [[SKYContainer defaultContainer] publicCloudDatabase];
[publicDB saveRecord:todo completion:^(SKYRecord *record, NSError *error) {
    if (error) {
        // Here you have an error object, which means the saving operation is failed.
        // you can get an error message by this:
        NSString *errorString = error.userInfo[SKYErrorMessageKey];
        // The above error message is returned from skygear server, and is intended for developer.

        // If you need a string to display to user, you can use `-[NSError localizedDescription]`
        NSString *errorString = [error localizedDescription];
        // For example: "You are not allowed to perform this operation."


        // You can handle the error even better by providing custom error message based on the context, and error code.
        if ([error code] == SKYErrorAccessTokenNotAccepted) {
            UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"Error occurred"
                                                                           message:@"You are logged out remotely"
                                                                    preferredStyle:UIAlertControllerStyleAlert];
            [alert addAction:[UIAlertAction actionWithTitle:@"OK"
                                                      style:UIAlertActionStyleDefault
                                                    handler:nil]];
            [self presentViewController:alert
                               animated:YES
                             completion:nil];
        }
        return;
    }
}];
```

```swift
let publicDB = SKYContainer.defaultContainer().publicCloudDatabase
publicDB.saveRecord(record) { (record : SKYRecord!, error : NSError!) in
    if let error = error as? SKYError {
        // Here you have an error object, which means the saving operation is failed.
        // You can get an error message by this:
        let userFriendlyString = error.userInfo[SKYErrorMessageKey] as! String
        // The above error message is returned from skygear server, and is intended for developer.

        // If you need a string to display to user, you can use `-[NSError localizedDescription]`
        let errorMessage = error.localizedDescription
        // For example: "You are not allowed to perform this operation."

        // You can handle the error even better by providing custom error message based on the context and error code.
        if error.code == SKYError.accessTokenNotAccepted {
            let alert = UIAlertController(title: "Error",
                                          message: "You are logged out remotely",
                                          preferredStyle: UIAlertControllerStyle.Alert)
            alert.addAction(UIAlertAction(title: "OK",
                                          style: UIAlertActionStyle.Default,
                                          handler: nil))
            self.presentViewController(alert,
                                       animated: true,
                                       completion: nil)
        }
    }
}
```

## Error codes

Detailed error code can be found in [here](https://github.com/SkygearIO/skygear-SDK-iOS/blob/master/Pod/Classes/SKYError.h).
Each of them concisely describe the error, you should compare error against
error codes if you want to handle them manually.
