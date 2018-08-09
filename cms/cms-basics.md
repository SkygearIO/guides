---
title: CMS Quick Start
---
[[toc]]

## Introduction

Skygear CMS aims to provide a user friendly-interface for business user to update the content of the app. As a developer, you can customize the YAML file at the Developer Portal. Below are the 5 major CMS features:

1. **Content management**: Display, filter, sort, create, and update records, including relation records.
2. **User management**: Reset password for users, disable users and verify users.
3. **Import/Export**: Import records, and export records according to the filtered view. 
4. **File upload**: Upload files. (Coming soon)
5. **Push**: Broadcast push notifications to segmented users. (Coming soon)

## Getting started

1. Make sure the version of your Skygear app is v1.6.1+. If not, upgrade your app to the latest version in the Setting tab.  
2. Enable the CMS plug-in. If the CMS plug-in is already on before you upgrade the app, disable the plug-in and reanble it. You will see a YAML editor when the CMS is enabled successfully.
3. Create an admin user on the right hand corner to login to the CMS.

![CMS overview](/assets/cms/cms-overview.png)

:::tips
If you are not familiar with YAML, we suggest you reading the [YAML doc](http://yaml.org/start.html) to familiarize yourself with the syntax. 
:::

## Configuring CMS pages

![CMS page types](/assets/cms/cms-page-types.png)

### Page types

Skygear CMS is consisted of **pages**. All pages will be shown on the left side bar so that CMS users can navigate across pages. Each page is configurable. You can configure various items depending on the page type.

There are four page types:

#### [Record][doc-cms-record]

You can manage the content of your app with record pages. Each record page can display a CMS record, and each CMS record is linked to a database table.

A record page has 4 views: **list**, **show**, **edit** and **new**. Simply put, they are the ways your business users read, update or create a record in the CMS.

Let say we have a blog app, and we want to let business users to update or create blog posts with the CMS. To do so, first of all, we should create a CMS record that links to the blog post table in the database, and then create a page for that CMS record.

#### [User management][doc-cms-user]

You can manage your users with the user management page. Current functions include assigning admin role and resetting passwords.

##### Push notifications
Coming soon.

#### File upload
Coming soon.

### Example

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

Suppose we want to show all the blogpost records and all the comment records in the CMS, and also need the user management dashboard:

```yml
site:
  - type: user_management
    label: User management
  - type: record
    name: blogpost  # this is the CMS record name, which can be configured below
    label: Blogposts
  - type: record
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

Next, let's learn about how to [configure the record page][doc-cms-record].


[doc-cms-record]: /guides/cms/cms-records/
[doc-cms-user]: /guides/cms/cms-user/