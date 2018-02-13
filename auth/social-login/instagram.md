---
title: User login with Instagram
---

[[toc]]

This article teaches you how to setup Instagram login in your application.

## Setup Instagram application

1. Click register your application in [Instagram developer portal][instagram-deveopers].

    ![Instagram developers portal](/assets/sso/instagram-developers.png)

2. Create client id. Make sure **Valid redirect URIs** contains following URL.
   Replace `${APP_NAME}` with your app name. 

    ```
    https://${APP_NAME}.skygeario.com/sso/instagram/auth_handler
    ```   

    ![Instagram create client id](/assets/sso/instagram-create-client-id.png)

3. Save your **Client ID** and **Client Secret**.

    ![Instagram obtain client id](/assets/sso/instagram-obtaion-client-id.png)

4. Go to Skygear portal, User Auth > Social Login. Enable Instagram in social connections and input your **Client ID** and **Client Secret**.

    ![Instagram Skygear portal](/assets/sso/instagram-portal.png)

5. **Optional**: Select the default permissions from Skygear portal, it will be used when user login
   with Skygear SDK. You can still overwrite it by providing another set of scope in the SDK call.

## Login with Instagram through Skygear SDK

- [JS][sso-js]
- [iOS][sso-ios]
- [Android][sso-android]

[instagram-deveopers]: http://instagram.com/developer/
[sso-js]: /guides/auth/social-login/js/#login-with-3rd-party-account
[sso-android]: /guides/auth/social-login/android/#login-with-3rd-party-account
[sso-ios]: /guides/auth/social-login/ios/#login-with-3rd-party-account