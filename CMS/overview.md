---
title: CMS Overview
---

## Introduction

Skygear CMS aims to provide a user friendly-interface for business user to update the content of the app. As a developer, you can configure what to display / be editable in the CMS with the YML file at the Developer Portal.

Some of you may wonder what's the difference between the CMS and the data browser, here's a summary.


|Difference	|Database Browser	|CMS	|
|---	|---	|---	|
|Target user	|Developers who need to view and alter database data and schema.	|System Admin or managers who need to view and update content of the system.	|
|Data views	|Showing all tables, including Skygear default tables such as _auth, _role ,etc. Also shows every fields including default fields in each table, such as _access, _id etc.	|Default hide all tables. Developers need to configure which table and fields to be shown at	|
|Data export	|Yes	|Yes	|
|Data import |No |Yes |
|Who can access	|Collaborators of your app on Skygear	|Admin users in Skygear user role record	|
|Customizable	|No. It default shows all tables and columns.	|Yes, with config in YML format.	|

This guide will give you an overview of the features of the CMS. If you want to learn about how to configure the CMS, read [CMS basics]().

## Core features

There are four core features the CMS provides. We will explain them in details below.

1. **Content management**: Display, filter, sort, create, update and delete records, including relation records.

2. **File upload**: Upload files

3. **Import/Export**: Import records, and export records according to the filtered view.

4. **Push**: Broadcast push notifications to segmented users. (Coming soon)

5. **User management**: Give CMS access to users, create a new user account, block a problematic user and etc. (Coming soon)

## Content management

Display, filter, sort, create, update and delete records, including relation records.

### Display records
