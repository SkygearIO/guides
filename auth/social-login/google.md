---
title: User login with Google
---

[[toc]]

This article teaches you how to setup Google login in your application.

## Setup Google project

1. Log in Google account and select or create project in [Google developers console][google-developers].

    ![Google select project](/assets/sso/google-select-project.png)
    
2. In credentials, create OAuth client ID

    ![Google create oauth client](/assets/sso/google-create-oauth-client.png)

3. Complete create client ID form. Make sure **Authorized redirect URIs** contains following link.
   Replace `${APP_NAME}` with your app name.

    ```
    https://${APP_NAME}.skygeario.com/sso/google/auth_handler
    ```

    ![Google authorized redirect uris](/assets/sso/google-authorized-redirect-uris.png)

4. Save your **Client ID** and **Client Secret** from Google developers console.

    ![Google client id and secret](/assets/sso/google-client-id-and-secret.png)

6. Go to Skygear portal, User Auth > Social Login. Enable Google in social connections and input your **Client ID** and **Client Secret**.

    ![Google Skygear portal](/assets/sso/google-portal.png)

7. Select the default permissions from Skygear portal, it will be used when user login
   with Skygear SDK. You can still overwrite it by providing another set of scope in the SDK call.
   Scope is required in google login, please select at lease one permission in your setup.

## Login with Google through Skygear SDK

- [JS][sso-js]
- [iOS][sso-ios]
- [Android][sso-android]


[google-developers]: https://console.developers.google.com/projectselector/apis/credentials
[sso-js]: /guides/auth/social-login/js/#login-with-3rd-party-account
[sso-android]: /guides/auth/social-login/android/#login-with-3rd-party-account
[sso-ios]: /guides/auth/social-login/ios/#login-with-3rd-party-account