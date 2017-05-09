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

```Javascript
skygear.publicDB.save(new Note({
  'content': 'Hello World!'
})).then((record) => {
  // save succeed
}, (error) => {
  // Here you have an error, the error object looks like this
  // { status: 401,
  //   error:
  //    { name: 'AccessTokenNotAccepted',
  //      code: 104,
  //      message: 'token does not exist or it has expired' } }

  // `error.error.message` is a message describing why an error occurred,
  // it is intended for developers to read.
  // If you want an user-friendly message for display, you can use:
  error.error.toLocaleString()
  // An example will be "You are not allowed to perform this operation."

  // You can handle the error even better by checking the error code and
  // notify users with your own error message.
  if (error.error.code === skygear.ErrorCodes.AccessTokenNotAccepted) {
    alert("You have been logged out remotely");
  }
});
```

## Error codes

Detailed error code can be found in [here](https://github.com/SkygearIO/skygear-SDK-JS/blob/master/lib/error.js).
Each of them concisely describe the error, you should compare error against
error codes if you want to handle them manually.
