---
title: CMS basics
---
[[toc]]

## Introduction

Skygear CMS aims to provide a user friendly-interface for business user to update the content of the app. As a developer, you can customize the YAML file at the Developer Portal. Below are the 5 major CMS features:

### Content management
Display, filter, sort, create, and update records, including relation records.

### Push
Broadcast push notifications to segmented users.

### User management
Give CMS access to users and reset user password.

### Import/Export
Import records, and export records according to the filtered view. 

### File upload
Upload files (Coming soon)


## Getting started

In this guide, you will learn how to configure the CMS with a YAML file. To start with, enable the CMS plug-in at the Developer Portal. You will see a YAML editor when the CMS is enabled successfully.

The new CMS is only available for Skygear v1.6.1+. If you are not a new Skygear user and you want to use the new CMS, upgrade your Skygear app to v1.6.1 and re-enable the CMS plug-in.

![CMS YML editor](/assets/cms/cms-yml-editor.png)

If you are not familiar with YAML, we suggest you reading the [YAML doc](http://yaml.org/start.html) to familiarize yourself with the syntax. 

## CMS overview

![CMS page types](/assets/cms/cms-page-types.png)

Skygear CMS is consisted of **pages**. All pages will be shown on the left side bar so that CMS users can navigate across pages. Each page is configurable. You can configure various items depending on the page type.

There are four page types:

### [Record][doc-cms-record]

You can manage the content of your app with record pages. Each record page can display a CMS record, and each CMS record is linked to a database table.

A record page has 4 views: **list**, **show**, **edit** and **new**. Simply put, they are the ways your business users read, update or create a record in the CMS.

Let say we have a blog app, and we want to let business users to update or create blog posts with the CMS. To do so, first of all, we should create a CMS record that links to the blog post table in the database, and then create a page for that CMS record.

### [Push notification][doc-cms-push]

You can send push messages to your app users with the push notification page.

### [User management][doc-cms-user]

You can manage your users with the user management page. Current functions include assigning admin role and resetting passwords.

### File upload
Coming soon.

Below is a high level overview of all the configurable items:

```YML
site:
  # pages configuration

records:
  # CMS record configuration

association_records:
  # Many-to-many record configuration

push_notifications:
  # push notification page configuration
```


## Configuring pages

Skygear CMS is consisted of pages. When working with the CMS configuration, the first things you should consider is what pages you would need in the CMS.

There are 3 page types: Record, Push Notification, and User Management. You can learn about what each type does in the [CMS overview](./#cms-overview) section.

To show you how to configure the pages, let's assume we have a database like this:

**Blogpost**
|Fields| Type| 
|---|---|
|_id| String|
|_created_at| DateTime|
|_updated_at| DateTime|
|title| String|
|content| String|
|cover| Asset|

**Comment**
|Fields| Type| 
|---|---|
|_id| String|
|_created_at| DateTime|
|_updated_at| DateTime|
|content| String|
|blogpostId | Reference|

Suppose we want to show all the blogpost records and all the comment records in the CMS, and also need the push notification dashboard and the user management dashboard:

```yml
site:
  - type: UserManagement
    label: User management
  - type: PushNotifications
    label: Push
  - type: Record
    name: blogpost  # this is the CMS record name, which can be configured below
    label: Blogposts
  - type: Record
    name: comment
    label: Comments

records:
  blogpost: # this is the CMS record name, you can call it anything
    record_type: blogpost  # this is the database table that is linked to this CMS record.
  comment:
    record_type: comment
```

 - `name` refers to the CMS record name. You can configure the CMS record name in the records section. 
 - `label` refers to the name to be shown in the CMS. If you don't provide a label, Skygear CMS will simply show the CMS record name.

Now save the changes and open the CMS. You should see all the pages on the left side bar.

![CMS Site](/assets/cms/cms-site.png)

## What's next?

Next, let's learn about how to [configure the Record page][doc-cms-record].


[doc-cms-record]: /guides/cms/cms-records/
[doc-cms-push]: /guides/cms/cms=push/
[doc-cms-user]: /guides/cms/cms-user/