---
title: Authenticating Users
---

[[toc]]


## Getting the current user ID

In your cloud code, you can obtain the user ID of the authenticated user by
calling the function `current_user_id` in the `skygear.utils.context` module.

The current user ID will be useful in database hooks, lambda functions, or
custom HTTP endpoint (if authenticated). Scheduled tasks are not run with
any authenticated user so you cannot get an authenticated user from there.


```javascript
skygearCloud.op('create-task', (param) => {
	const user_id = 
}, {
	userRequired: true
});
```


## Reset Password

Not implemented in JS.
