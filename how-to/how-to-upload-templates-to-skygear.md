# How to upload templates to Skygear

## Introduction

Almost every software application that requires users to log in comes with the ability to send password reset and welcome emails. Landing pages for user verification have also become an essential for a secure and complete user registration flow.

On Skygear, these features are all implemented and brought to users out of the box. At the same time one can customize the appearance and behaviours of these emails and landing pages through the use of templates.

## Prerequisites

Before you start this section, please ensure you have `skycli` installed and configured properly. If not, follow [this](../set-up/set-up-steps.md) to set it up.

## Available templates

Here is a list of customizable emails and pages:

* _forgot\_password\_email.txt_
* _forgot\_password\_email.html_
* _forgot\_password\_reset.html_
* _forgot\_password\_success.html_
* _forgot\_password\_error.html_
* _welcome\_email.txt_
* _welcome\_email.html_
* _user\_verification\_general\_error.html_
* _user\_verification\_message.txt_
* _user\_verification\_message.html_
* _user\_verification\_success.html_
* _user\_verification\_error.html_

In this guide, we will create and upload a pair of forgot password templates, in html and txt format.

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

To trigger Auth gear to send a forgot password email, you can either call the `requestForgotPasswordEmail` method in Skygear SDK or directly make a HTTP-based API call. Here we will go for the latter with the help of curl. Replace the placeholders with your Skygear app's data:

```text
curl -X POST \
  https://<your-app>.skygearapp.com/_auth/forgot_password \
  -H 'Cache-Control: no-cache' \
  -H 'Connection: keep-alive' \
  -H 'Content-Length: 33' \
  -H 'accept: */*' \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -H 'sec-fetch-mode: cors' \
  -H 'sec-fetch-site: same-site' \
  -H 'x-skygear-api-key: <your_app_key>' \
  -d '{"email":<a_registered_email>}'
```

\(\_\_TODO\_\_ skygear endpoint\)

Be sure that the email address you put in the JSON body is one that has been registered before on your app. Otherwise an error saying no such email address found will be thrown.

Once you get the OK response back, you can go ahead to your email inbox and check out the email created from your forgot password templates.

Note that you may only see the HTML part if you are using Gmail. In this case, you can show the email as original to look at the text part.

## Conclusion

Now you have the idea on how templates can be easily configured on Skygear. Other ones can be set in similar ways, just make sure you spell the template names correctly.

