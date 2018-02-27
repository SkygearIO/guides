---
title: User login with Facebook
---

[[toc]]

This article teaches you how to setup Facebook login in your application.

## Setup Facebook application

1. Create your Facebook application in [Facebook developer portal][facebook-deveopers].

    ![Facebook developers](/assets/sso/facebook-developers.png)

    ![Facebook create application](/assets/sso/facebook-create-application.png)

2. Add Facebook login in product.

    ![Facebook add login](/assets/sso/facebook-add-login.png)

3. In Facebook login setting, add following URL to **Valid OAuth redirect URIs**.
   Replace `${APP_NAME}` with your app name. 

    ```
    https://${APP_NAME}.skygeario.com/sso/facebook/auth_handler
    ```

    ![Facebook config redirect uris](/assets/sso/facebook-redirect-uris.png)

4. Go to Settings > Basic to make your application public.

5. Save your **App ID** and **App Secret** from Facebook portal.

    ![Facebook app id and secret](/assets/sso/facebook-app-id-and-secret.png)

6. Go to Skygear portal, User Auth > Social Login. Enable Facebook in social connections and input your **App ID** and **App Secret**.

    ![Facebook Skygear portal](/assets/sso/facebook-portal.png)

7. **Optional**: Select the default permissions from Skygear portal, it will be used when user login
   with Skygear SDK. You can still overwrite it by providing another set of scope in the SDK call.


## Login with Facebook through Skygear SDK

- [JS][sso-js]
- [iOS][sso-ios]
- [Android][sso-android]

[facebook-deveopers]: https://developers.facebook.com/
[sso-js]: /guides/auth/social-login/js/#login-with-3rd-party-account
[sso-android]: /guides/auth/social-login/android/#login-with-3rd-party-account
[sso-ios]: /guides/auth/social-login/ios/#login-with-3rd-party-account