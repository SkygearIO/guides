---
title: Social Login
---

[[toc]]


## Social Login (Facebook, Twitter, etc)

Besides the traditional login method using a password,
Skygear also supports authenticating a user with third party services
such as Facebook and Twitter, using an
[auth provider][doc-auth-provider] written with cloud code.

You can use the [`skygear.auth.loginWithProvider`](https://docs.skygear.io/js/reference/v1/class/packages/skygear-core/lib/auth.js~AuthContainer.html#instance-method-loginWithProvider) method by providing the name of the
auth provider, along with the relevant auth data (an object) necessary for
the authentication, such as the access token obtained from Facebook.

For example, if you have implemented a Facebook auth provider under the name
`com.facebook` using
`@skygear.provides('auth', 'com.facebook')` in the cloud code,
you can log a user in by:

``` javascript
const provider = 'com.facebook';
const authData = {
  'access_token': 'abcdefg',
};
skygear.auth.loginWithProvider(provider, authData);
```

- `provider`: name of the provider, such as `com.facebook` or `com.google`
- `authData`: an object that will be passed as a dictionary to the
  plugin that contains the information necessary to authenticate a user

Note: There is no distinction between signing up and logging in with
an auth provider. the Skygear server will create a new user if the principal ID
(as returned by the auth provider) does not exist in the database.

[doc-auth-provider]: /guides/advanced/auth-provider/python/
