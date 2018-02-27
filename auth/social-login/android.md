---
title: Social Login
description: User login with 3rd party authentication in Android.
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

- [Login with web flow][login-with-web-flow]
- [Login with access token][login-with-access-token]

## Login with web flow

1. Configure Callback URLs

    The callback URL is URL of your application that Skygear redirects to after user is authenticated.  You can configure the app scheme by adding the following lines into `strings.xml` and replace `${YOUR_APP_SCHEME}` with your URL scheme.

    ```xml
    <string name="skygear_scheme">${YOUR_APP_SCHEME}</string>
    ```

    Skygear SDK will redirect back to your application after user is authenticated through the following link. For production app, please make sure your Skygear portal *Allow redirect URLs* contains the below link. All URLs will be allowed, if *Allow redirect URLs* is empty.

    ```
    ${YOUR_APP_SCHEME}://skygeario.com/auth_handler
    ```

    ![Allow redirect URLs](/assets/common/sso-allow-redirect-urls.png)

2. Trigger login with web flow

    By calling `loginOAuthProvider`, Skygear SDK will complete the login flow by using browser to login user.

    ```java
    import io.skygear.skygear.sso.OAuthOption;
    import io.skygear.skygear.sso.OAuthOptionBuilder;

    String providerID = "google";

    // Optional: overwrite the portal default permissions if it is provided
    String[] scope = ["email"]; 

    OAuthOption options = new OAuthOptionBuilder()
        .setScheme(${YOUR_APP_SCHEME})
        .setScope(scope) // optional
        .getOption();

    skygear.getAuth().loginOAuthProvider(provider, options, activityContext, new AuthResponseHandler() {
        @Override
        public void onAuthSuccess(Record user) {
            Log.i("Skygear SSO", "onAuthSuccess: Got token: " + user.getAccessToken());
        }

        @Override
        public void onAuthFail(Error error) {
            Log.e("Skygear SSO", "onAuthFail: Reason: " + error.getMessage());
        }
    });
    ```

## Login with access token

If you obtain the access token from 3rd party provider SDK, you can login to provider as follows:

```java
String providerID = "google";
String accessToken = "access token from 3rd party sdk";

skygear.getAuth().loginOAuthProviderWithAccessToken(providerID, accessToken, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(Record user) {
        Log.i("Skygear SSO", "onAuthSuccess: Got token: " + user.getAccessToken());
    }

    @Override
    public void onAuthFail(Error error) {
        Log.e("Skygear SSO", "onAuthFail: Reason: " + error.getMessage());
    }
});
```

## Link current user with web login flow

Follow steps in [Login with web flow][login-with-web-flow] and call `linkOAuthProvider` instead of `loginOAuthProvider` after user login.

```java
import io.skygear.skygear.sso.OAuthOption;
import io.skygear.skygear.sso.OAuthOptionBuilder;

String providerID = "google";

// Optional: overwrite the portal default permissions if it is provided
String[] scope = ["email"];

OAuthOption options = new OAuthOptionBuilder()
    .setScheme(${YOUR_APP_SCHEME})
    .setScope(scope) // optional
    .getOption();

skygear.getAuth().linkOAuthProvider(providerID, options, activityContext, new LinkProviderResponseHandler() {
    @Override
    public void onSuccess() {
        Log.i("Skygear SSO", Link successfully");
    }

    @Override
    public void onFail(Error error) {
        Log.e("Skygear SSO", "Fail to link: Reason: " + error.getMessage());
    }
});
```

## Link current user with access token

If you obtain the access token from 3rd party provider SDK, you can link the current user to provider as follows:

```java
String providerID = "google";
String accessToken = "access token from 3rd party sdk";

skygear.getAuth().linkOAuthProviderWithAccessToken(providerID, accessToken, new LinkProviderResponseHandler() {
    @Override
    public void onSuccess() {
        Log.i("Skygear SSO", Link successfully");
    }

    @Override
    public void onFail(Error error) {
        Log.e("Skygear SSO", "Fail to link: Reason: " + error.getMessage());
    }
});
```

## Unlink current user with provider

To unlink current user with the provider.

```java
String providerID = "google";

skygear.getAuth().unlinkOAuthProvider(providerID, new UnlinkProviderResponseHandler() {
    @Override
    public void onSuccess() {
        Log.i("Skygear SSO", Unlink successfully");
    }

    @Override
    public void onFail(Error error) {
        Log.e("Skygear SSO", "Fail to unlink: Reason: " + error.getMessage());
    }
});
```

## Get the 3rd party provider user profiles

`getOAuthProviderProfiles` will return a JSON object of user connected provider profiles. The keys are the providers ID and the values are the profile JSON object. Every time when user login, Skygear will request and update the 3rd party user profile. This is useful to identify if user links with provider.

```java
skygear.getAuth().getOAuthProviderProfiles(new GetOAuthProviderProfilesResponseHandler() {
    @Override
    public void onSuccess(JSONObject result) {
        Log.i("Skygear SSO", "Link successfully: Profiles: " + result.toString());

        if (result.has("google")) {
            Log.i("Skygear SSO", "User has linked with google");
        }
    }

    @Override
    public void onFail(Error error) {
        Log.e("Skygear SSO", "Fail to get profiles: Reason: " + error.getMessage());
    }
});
```

[login-with-web-flow]: #login-with-web-flow
[login-with-access-token]: #login-with-access-token
[facebook]: /guides/auth/social-login/facebook/
[google]: /guides/auth/social-login/google/
[linkedin]: /guides/auth/social-login/linkedin/
[instagram]: /guides/auth/social-login/instagram/
