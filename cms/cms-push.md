---
title: Configuring the Push Dashboard
---

[[toc]]

## Introduction

Push dashboard allows your admins to broadcast push notifications to your users. You can also segment the users so that you can send targeted push notifications.

![](/assets/cms/cms-push.png)

Make sure you have enabled Skygear push notification and fill in the information accordingly so that the push dashboard can work properly.

![](/assets/cms/push-notification-portal.png)


## Configuring the push dashboard

Example usage:

```yml
site:
  - type: PushNotifications
    label: Push Dashboard

push_notifications:
  filters:
    - name: username
      type: String
      label: Username
```

`fiters`: It refers to the fields admins can use to filter the users. For example, if you set the `username` field to be filterable, then admins can filter users based on their username and send push notifications to the filtered user list.

Below are the filter type you can use:

- String
- DateTime
- Boolean
- Integer
- General
- Reference