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

Not implemented in JS.
