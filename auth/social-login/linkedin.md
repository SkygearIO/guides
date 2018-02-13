---
title: User login with LinkedIn
---

[[toc]]

This article teaches you how to setup LinkedIn login in your application.

## Setup LinkedIn application

1. Create your LinkedIn application in [LinkedIn developer portal][linkedin-deveopers].

    ![LinkedIn create application](/assets/sso/linkedin-create-application.png)

2. Complete create application form.

3. In LinkedIn application settings > authentications, add following URL to **Authorized redirect URLs**.
   Replace `${APP_NAME}` with your app name. Save your **Client ID** and **Client Secret**.

    ```
    https://${APP_NAME}.skygeario.com/sso/linkedin/auth_handler
    ```

    ![LinkedIn authorized redirect urls](/assets/sso/linkedin-authorized-redirect-urls.png)

4. Go to Skygear portal, User Auth > Social Login. Enable LinkedIn in social connections and input your **Client ID** and **Client Secret**.

    ![LinkedIn Skygear Portal](/assets/sso/linkedin-portal.png)

5. **Optional**: Select the default permissions from Skygear portal, it will be used when user login
   with Skygear SDK. You can still overwrite it by providing another set of scope in the SDK call.

## Login with LinkedIn through Skygear SDK

- [JS][sso-js]
- [iOS][sso-ios]
- [Android][sso-android]

[linkedin-deveopers]: https://www.linkedin.com/developer/apps
[sso-js]: /guides/auth/social-login/js/#login-with-3rd-party-account
[sso-android]: /guides/auth/social-login/android/#login-with-3rd-party-account
[sso-ios]: /guides/auth/social-login/ios/#login-with-3rd-party-account