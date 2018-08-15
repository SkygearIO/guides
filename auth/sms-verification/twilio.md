---
title: Sending verification SMS via Twilio
---

[[toc]]

This article teaches you how to setup Twilio for sending verification SMS.

1. Sign up account in [Twilio][twilio] and create a project.

2. Obtain *Account SID* and *Auth Token* from Twilio General Settings.

    ![Twilio General Settings](/assets/user-verification/twilio-general-settings.png)

::: note

Using test credentials can test API requests to send SMS messages without
charging your Twilio account or sending an SMS. [Learn more][twilio-test-sms-messages].

:::

3. Input *Account SID*, *Auth token* and *Sender Phone Number* in Skygear portal.

    ![Twilio Skygear Portal](/assets/user-verification/skygear-portal-twilio-screenshot.png)

## Verify user with code API

At this point, your user will be able to obtain verification code via SMS. To
verify user with code in your application:

- [JS][sms-verification-js]
- [iOS][sms-verification-ios]
- [Android][sms-verification-android]

[twilio]: https://www.twilio.com/
[twilio-test-sms-messages]: https://www.twilio.com/docs/iam/test-credentials#test-sms-messages
[sms-verification-js]: /guides/auth/sms-verification/js/#verify-user-with-code-api
[sms-verification-android]: /guides/auth/sms-verification/android/#verify-user-with-code-api
[sms-verification-ios]: /guides/auth/sms-verification/ios/#verify-user-with-code-api
