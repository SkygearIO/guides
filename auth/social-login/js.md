---
title: Social Login
description: User login with 3rd party authentication in web.
---

[[toc]]

Skygear allows you to authenticate users using their 3rd party accounts.

## Setup

Currently following providers are supported. Please find the setup instructions below:

- [Google][google]
- [Facebook][facebook]
- [LinkedIn][linkedin]
- [Instagram][instagram]

### Login with 3rd party account

Skygear SDK supports two kinds of login flow:

- [Login with Skygear SDK (Web only)][login-with-skygear-sdk]
- [Login with access token][login-with-access-token]

## Login with Skygear SDK (Web only)

1. Start the login flow either by prompting the pop-up window or by redirecting to sign-in page.
    - Login with popup window

    ```javascript
    skygear.auth.loginOAuthProviderWithPopup('google').then(function(user) {
      console.info('Login success', user);

      console.info('Access token:', skygear.auth.accessToken);
      console.info('Username:', skygear.auth.currentUser.username);
    }, function(error) {
      console.error('Login fail', error);
    });
    ```

    - Login by redirecting to the sign-in page

    ```javascript
    skygear.auth.loginOAuthProviderWithRedirect('google');
    ```

    After calling `loginOAuthProviderWithRedirect`, user will be redirected to the 3rd party auth flow.
    Then you can retrieve the auth result by `getLoginRedirectResult` when your page loads.

    ```javascript
    skygear.auth.getLoginRedirectResult().then(function (user) {
      if (user) {
        console.info('Login success', user);

        console.info('Access token:', skygear.auth.accessToken);
        console.info('Username:', skygear.auth.currentUser.username);
      } else {
        console.info('No redirect operation was called');
      }
    }, function(error) {
      console.error('Fail to retrieve redirect result', error);
    });
    ```

    **Optional**: If you want your user lands on another page rather than the sign-up page after authentication,
    you can call `loginOAuthProviderWithRedirect` this way:

    ```javascript
    skygear.auth.loginOAuthProviderWithRedirect('google', { callbackUrl: "${CALLBACK_URL}" });
    ```

    **Optional**: To define scopes in login call, you can overwirte the default permission which has been set in portal.

    ```javascript
    // popup window
    skygear.auth.loginOAuthProviderWithPopup('google', { scopes: ['email'] });

    // redirect
    skygear.auth.loginOAuthProviderWithRedirect('google', { scopes: ['email'] });
    ```

2. For production app, please make sure your skygear portal *Allow redirect URLs* contains your sign-up page URL or the callback URL that you specified. All URLs will be allowed, if *Allow redirect URLs* is empty.

    ![Allow redirect URLs](/assets/common/sso-allow-redirect-urls.png)

## Login with access token

If you obtain the access token from 3rd party provider SDK, you can login to provider as follows:

```javascript
skygear.auth.loginOAuthProviderWithAccessToken('google', accessToken).then(function (result) {
  console.info('Login success', user);

  console.info('Access token:', skygear.auth.accessToken);
  console.info('Username:', skygear.auth.currentUser.username);
}).catch(function (err) {
  console.error('Login fail', error);
});
```

## Link with Skygear SDK (Web only)

Similar to [Login with web flow][login-with-skygear-sdk], calling `linkOAuthProviderWithPopup`
and `linkOAuthProviderWithRedirect` instead.

- Link with popup window

```javascript
skygear.auth.linkOAuthProviderWithPopup('google').then(function() {
  console.info('Link provider success');
}, function(error) {
  console.error('Link provider fail', error);
});
```

- Login by redirecting to the sign-in page

```javascript
skygear.auth.linkOAuthProviderWithRedirect('google');
```

After calling `loginOAuthProviderWithRedirect`, user will be redirected to the 3rd party auth flow.
Then you can retrieve the auth result by `getLinkRedirectResult` when your page loads.

```javascript
skygear.auth.getLinkRedirectResult().then(function (result) {
  if (result) {
    console.info('Link provider success');
  } else {
    console.info('No redirect operation was called');
  }
}, function(error) {
  console.error('Fail to retrieve redirect result', error);
});
```

## Link current user with access token

If you obtain the access token from 3rd party provider SDK, you can link the current user to provider as follows:

```javascript
skygear.auth.linkOAuthProviderWithAccessToken('google', accessToken)
.then(function() {
  console.info('Link provider success');
}).catch(function (err) {
  console.error('Link provider fail', error);
});
```

## Unlink current user with provider

To unlink current user with the provider.

```javascript
skygear.auth.unlinkOAuthProvider('google')
.then(function() {
  console.info('Unlink provider success');
}).catch(function (err) {
  console.error('Unlink provider fail', error);
});
```

## Get the 3rd party provider user profiles

`getOAuthProviderProfiles` will return an object of user connected provider profiles. The keys are the providers ID and the values are the profile object. Every time when user login, Skygear will request and update the 3rd party user profile. This is useful to identify if user links with provider.

```javascript
skygear.auth.getOAuthProviderProfiles().then(function (result) {
  console.info('Get provider profiles success', result);
  if (result['google']) {
    console.info('User has linked with google');
  }
}, function (error) {
  console.error('Get provider profiles fail', error);
});
```

[login-with-skygear-sdk]: #login-with-skygear-sdk
[login-with-access-token]: #login-with-access-token
[facebook]: /guides/auth/social-login/facebook/
[google]: /guides/auth/social-login/google/
[linkedin]: /guides/auth/social-login/linkedin/
[instagram]: /guides/auth/social-login/instagram/
