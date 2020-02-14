# Template Management

## Introduction

Almost every software application that requires users to log in comes with the ability to send password reset and welcome emails. Landing pages for user verification have also become an essential for a secure and complete user registration flow.

In Skygear, these features are all implemented and brought to users out of the box. At the same time one can customize the appearance and behaviours of these emails and landing pages through the use of templates.

## Requirements

All user-defined templates must reside in a folder called `templates` next to `skygear.yaml`.

Some of them must be placed within a sub-folder named by corresponding [Login ID](../auth/user-and-identity/login-ids.md):

```text
templates[/<login_id>]/<template>
```

Say you have a self-defined Login ID called`company_email` whose type is a email address and you would like to upload a `user_verification_email.html` template for it. You will have to place this template under the path`templates/company_email/user_verification_email.html`, otherwise it will be marked as invalid and the upload will fail.

## Operations on Templates

Currently there are three possible operations on templates. You can list uploaded ones, download them or upload new ones with these `skycli` [commands](list-of-commands.md#skycli-app-list-templates).

## Available Templates

#### `welcome_email.html`

Customization on the greeting message sent to users via HTML email when s/he signs up is done with this template.

#### `welcome_email.txt`

The template the decides how the welcome email in plain text looks like.

#### `forgot_password_email.html`

This template will be used to generate the HTML version of all forgot password emails.

#### `forgot_password_email.txt`

All plain text version of forgot password emails will follow this template.

#### `forgot_password_error.html`

This will be displayed when a user encounters error during the reset password process.

#### `forgot_password_reset.html`

This will be the HTML page a user uses to reset his/her password.

#### `forgot_password_success.html`

The page a user will be redirected to upon successful password reset attempt.

#### `mfa_oob_code_email.html`

The HTML version of email carrying the Out of Band \(OOB\) code for MFA which will be sent to a user's email. 

#### `mfa_oob_code_email.txt`

The plain text counterpart of the above template.

#### `mfa_oob_code_sms.txt`

The template used to generated SMS messages sent to users carrying the OOB code for MFA.

#### `user_verification_email.html *`

This template generates HTML emails that will be sent to users to verify their email address for a Skygear app.

It has to be put under the sub-directory with a valid Login ID name whose type is email.

#### `user_verification_email.txt *`

The plain text version of the above template. Once again, remember to put it under a login-id-named directory.

#### `user_verification_success.html *`

Template responsible for user verification success page. Again, has to be in a login-id-named directory.

#### `user_verification_error.html *`

This template is used to customize what you would like the user see when the user verification process fails.

Needs a mother directory with a named of a valid Login ID.

#### `user_verification_sms.txt *`

This plain text based template decides what the SMS sent to verify a user of an app looks like. Has to be in a directory whose name is a phone number type Login ID.

## Tutorial on Uploading Templates

[Here](../how-to/how-to-upload-templates-to-skygear.md) is a step-by-step tutorial on how can templates be uploaded to Skygear.



