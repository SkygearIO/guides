---
title: Authenticating Users
---

[[toc]]


## Getting the current user ID

The current user ID are available from the `context` in the `options` of the Cloud Functions
callback. You can retrieve the user ID by:

```js
skygearCloud.op('foo', function (param, options) {
  const {
    context
  } = options;
  console.log(context.user_id);
}, {
  userRequired: true
});
```

## Reset Password

From v1.5 onward, you can reset password for you users using `skygear.auth.adminResetPassword(user, newPassword)`. To call this function, you must use the master key of your Skygear app.

Suppose we are writing a lambda function to be called by your admins at the client-side to reset password for a user.

```Javascript
const skygearCloud = require("skygear/cloud");

const container = skygearCloud.getContainer();
container.apiKey = skygearCloud.settings.masterKey;
container.endPoint = skygearCloud.settings.skygearEndpoint + '/';

skygearCloud.op('resetPasswordByAdmin', function(params){
  // assume the user id and the new password are supplied by the client-side
  let userId = params.args.userId
  let newPassword = params.args.newPassword
  /*
   we suggest you checking whether this lambda call is made by an admin 
   before calling adminResetPassword
   */
  container.auth.adminResetPassword(userId, newPassword)
    .then((user) => console.log(user))
    .catch((err) => console.log(err))
})
```
Here we have configured a Skygear cloud container with the endpoint and the master key of our app. Normally you need to [supply a user_id to the container](https://docs.skygear.io/guides/cloud-function/calling-skygear-api/js/#skygear-container) so that you can make Skygear API calls on behalf of that user. However in the case of resetting password, we do not need to do so because when `adminResetPassword` is called with the master key, Skygear will reset the password of the user on behalf of himself.