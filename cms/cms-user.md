---
title: Configuring the User Management Dashboard
---

[[toc]]

## Introduction

The user management dashboard is where admins can manage the users of the app. Currently the user management dashboard has 3 features.

- Reset password
- Set admins (once a user is set to be an admin, not only he will be given the access to the CMS, he also will be assigned the admin role i.e. hasRole = admin)
- Create new users

![Skygear CMS users](/assets/cms/cms-user.png)

## Configuring the user management dashboard

To enable the user management dashboard in the CMS is very simple, simply configure it in the site section.

```yml
site:
  - type: UserManagement
    label: User Management Dashboard
```

