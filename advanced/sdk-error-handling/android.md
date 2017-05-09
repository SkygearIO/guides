---
title: Error Handling in SDK
description: How to handle errors gracefully when something goes wrong
---

[[toc]]

# Error Handling

Whenever you perform an action that calls the server, there is chance the
server rejects your request for various reasons.
This guide demonstrate how should the errors be handled.

For example when you try to save a record to the database and without
logging in, you get an error.

```java
RecordSaveResponseHandler handler = new RecordSaveResponseHandler(){
    // Override other methods such as onSaveSuccess
    @Override
    public void onSaveFail(Error error) {
        // Here you have an error object, which means the saving operation is failed.
        // You can get an error message by this:
        error.getMessage();
        // The above error message is returned from skygear server, and is intended for developer.

        // If you need a string to display to user, you can use `.getLocalizedMessage()`
        error.getLocalizedMessage();
        // For example: "You are not allowed to perform this operation."

        // You can handle the error even better by providing custom error message based on the context and error code.
        if (error.getCode() == Error.Code.ACCESS_TOKEN_NOT_ACCEPTED) {
            Log.i("Skygear Record Save", "You are logged out remotely");
        }
    }
};
database.save(aNote, handler);
```

## Error codes

Detailed error code can be found in [here](https://github.com/SkygearIO/skygear-SDK-Android/blob/master/skygear/src/main/java/io/skygear/skygear/Error.java).
Each of them concisely describe the error, you should compare error against
error codes if you want to handle them manually.
