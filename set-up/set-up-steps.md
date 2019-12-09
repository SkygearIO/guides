---
description: >-
  skycli is a command line tool for developers to manage their Skygear apps,
  which is a prerequisite for all following sections.
---

# Install and Configure skycli

**1. Install `skycli`** **.**

```bash
$ npm install -g @skygear/skycli
# OR
$ yarn global add @skygear/skycli
```

**2. Select the cluster you need.** If you already are a skygear.io user or would like to try it out, select skygeario. If you are an Enterprise license user, select the other option and enter the cluster information you receive from our team. In this example, we will go for the first option.

```bash
$ skycli config set-cluster
❯ skygeario
  Connect to my own cluster
```

**3. Log in or Sign up to Skygear.** To create a new app or manage existing ones, you will need an account first. How an app is created will be covered later, here we will focus on logging in/signing up to Skygear.

Simply log in if you already have an account:

```bash
$ skycli auth login
? Email: <your_email>
? Password: [input is hidden]
Sign up as <your_email>.
```

Otherwise sign up to create a new one:

```bash
$ skycli auth signup
? Email: <your_email>
? Password: [input is hidden]
Sign up as <your_email>.
```

 Skygear decides whether you have access to apps by your account. Creator of an app is automatically granted access and the right to add more collaborators via sending invitation emails.

**4. Check configuration.** Type in the following command to check if you have connected `skycli`  to Skygear's cluster properly. Your output should look like the one displayed below.

```bash
$ skycli config view
┌──────────────────┬──────────────────────────────────────────┐
│ Property         │ Value                                    │
├──────────────────┼──────────────────────────────────────────┤
│ Cluster Type     │ cloud                                    │
├──────────────────┼──────────────────────────────────────────┤
│ Cluster Endpoint │ https://controller.skygear.dev           │
├──────────────────┼──────────────────────────────────────────┤
│ Cluster API Key  │ sq3GcUp0QVRTYPwBpnLcQi3jHK7SJFfF         │
├──────────────────┼──────────────────────────────────────────┤
│ Account          │ <your_email>                             │
└──────────────────┴──────────────────────────────────────────┘
```



