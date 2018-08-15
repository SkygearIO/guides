---
title: Sending verification SMS via Nexmo
---

[[toc]]

This article teaches you how to setup Nexmo for sending verification SMS.

1. Sign up account in [Nexmo][Nexmo] and create a project.

2. Obtain *API Key* and *API Secret* from Nexmo get started page.

    ![Nexmo Get Started](/assets/user-verification/nexmo-get-started.png)

3. Input *API Key*, *API Secret* and *Sender Phone Number* in Skygear portal.

    ![Nexmo Skygear Portal](/assets/user-verification/skygear-portal-nexmo-screenshot.png)

## Verify user with code API

At this point, your user will be able to obtain verification code via SMS. To
verify user with code in your application:

- [JS][sms-verification-js]
- [iOS][sms-verification-ios]
- [Android][sms-verification-android]

[nexmo]: https://www.nexmo.com/
[sms-verification-js]: /guides/auth/sms-verification/js/#verify-user-with-code-api
[sms-verification-android]: /guides/auth/sms-verification/android/#verify-user-with-code-api
[sms-verification-ios]: /guides/auth/sms-verification/ios/#verify-user-with-code-api
