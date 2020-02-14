# How to upload templates to Skygear

## Introduction

In this guide, we will create and upload a pair of forgot password templates, in html and txt format. They won't override each other as they server different purposes.

## Prerequisites

Before you start this section, please ensure you have `skycli` installed and configured properly. If not, follow [this](../set-up/set-up-steps.md) to set it up.

## Create some templates

All Skygear templates must be stored in a `templates` folder next to `skygear.yaml`. Let's start with creating one:

```text
$ mkdir templates && cd $_
```

Then create a html file called `forgot_password_email.html` and fill it in the following content:

{% code title="forgot\_password\_email.html" %}
```markup
<div>
    This is a forgot_password_email HTML template
</div>
```
{% endcode %}

Create another file called `forgot_password_email.txt`:

{% code title="forgot\_password\_email.txt" %}
```markup
This is a forgot_password_email text template
```
{% endcode %}

Right, we now have our forgot password email templates ready. We will proceed to uploading them.

## Upload the templates

Go back to the root folder where `skygear.yaml` lives. Then type in:

```text
$ skycli app update-templates
```

Both `forgot_password_email.txt` and `forgot_password_email.html` will be added. If you wish to know which ones are uploaded, enter:

```text
$ skycli app list-templates
```

A list of templates containing either their storage path or the message `<not provided>` will be shown.

## Check out how they look!

To reset a password, you first need a registered user. Call the `signup` method in Skygear SDK or directly `curl` the endpoint to create a user to your app. Here we will go for the latter option, be sure to replace all the placeholders wrapped in `<...>`:

```text
curl -X POST \
    https://<your-app>.skygearapp.com/_auth/signup \
    -H 'Cache-Control: no-cache' \
    -H 'Connection: keep-alive' \
    -H 'accept: */*' \
    -H 'cache-control: no-cache' \
    -H 'content-type: application/json' \
    -H 'sec-fetch-mode: cors' \
    -H 'sec-fetch-site: same-site' \
    -H 'x-skygear-api-key: <your_app_key>' \
    -d '{"login_ids": [{"key": "email", "value": "<an_email_address>" }], "password": "<some_password>" }'
```

Then you can trigger AuthGear to send a forgot password email to the newly registered email. Once again we will `curl` the endpoint, but of course you can do the same thing with the `requestForgotPasswordEmail` method in Skygear SDK:

```text
curl -X POST \
  https://<your-app>.skygearapp.com/_auth/forgot_password \
  -H 'Cache-Control: no-cache' \
  -H 'Connection: keep-alive' \
  -H 'accept: */*' \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -H 'sec-fetch-mode: cors' \
  -H 'sec-fetch-site: same-site' \
  -H 'x-skygear-api-key: <your_app_key>' \
  -d '{"email": "<a_registered_email>"}'
```

Be sure that the email address you put in the JSON body is one that has been registered before on your app. Otherwise an error saying no such email address found will be thrown.

Once you get the OK response back, you can go ahead to your email inbox and check out the email created from your forgot password templates.

Note that you may only see the HTML part if you are using Gmail, as this is their default display setting. In this case, you can show the email as original to look at the text part.

## Conclusion

Now you have the idea on how templates can be easily configured on Skygear. Other ones can be set in similar ways, just make sure you spell the template names correctly.

