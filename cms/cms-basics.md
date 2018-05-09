---
title: CMS basics
---
[[toc]]

## Introduction

Skygear CMS aims to provide a user friendly-interface for business user to update the content of the app. As a developer, you can customise the YML file at the Developer Portal. Below are the 5 major CMS features:

1. **Content management**: Display, filter, sort, create, update and delete records, including relation records.

2. **File upload**: Upload files

3. **Push**: Broadcast push notifications to segmented users.

4. **User management**: Give CMS access to users, create a new user account, reset user password and etc.

5. **Import/Export**: Import records, and export records according to the filtered view. (Coming soon)


Some of you may wonder what is the difference between the CMS and the data browser, here is a summary.

|Difference | Database Browser |CMS|
|----|----|----|
|Target User| Developers who need to view data and the database schema|Business users who need to view and update content of the system|
|Data views|Display all the tables at in database, including Skygear default tables such as _auth, _role ,etc. Display also every fields of a record, which includes default fields such as _access, _id etc.|Developers can configure which record type and its fields to be displayed |
|Data export|Yes |Yes, and you can configuration the exportable view|
|Data import|No|Yes|
|Who can access|Collaborators of your app on Skygear|Skygear users marked as admin|
|Customizable|No. It default shows all tables and columns.|Yes, with config in YML format.|


## Getting started

In this guide, you will learn how to configure the CMS with a YML file. To start with, enable the CMS plug-in at the Developer Portal. You will see a YML editor when the CMS is enabled successfully.

If you are not a new Skygear user and you want to use this new version of CMS, re-enable the CMS plug-in.

![CMS YML editor](/assets/cms/cms-yml-editor.png)


## CMS overview

![CMS page types](/assets/cms/cms-pages-types.png)

Skygear CMS is consisted of **pages**. All pages will be shown on the left side bar so that CMS users can navigate across pages. Each page is configurable. You can configure various items depending on the page type.

There are three page types:

**1. Record**

You can manage the content of your app with record pages. Each record page can display a CMS record, and each CMS record is linked to a database table.

A record page has 4 views: **list**, **show**, **edit** and **new**. Simply put, they are the ways your business users read, update or create a record in the CMS.

Let say we have a blog app, and we want to let business users to update or create blog posts with the CMS. To do so, first of all, we should create a CMS record that links to the blog post table in the database, and then create a page for that CMS record.

**2. Push notification**

You can send push messages to your app users with the push notification page.

**3. User management**

You can manage your users with the user management page. Current functions include assigning admin role and resetting passwords.

Below is a high level overview of all the configurable items:

```YML
site:
  # pages configuration

records:
  # CMS record configuration
  CMS_record_name:
    record_type:
    list:
      fields:
      filters:
      predicates:
      default_sort:
    show:
    edit:
    new:

user_management:
  # user management page configuration
  enable:

push_notifications:
  # push notification page configuration
  enable:

```

## Configuring pages

Skygear CMS is consisted of pages. When working with the CMS configuration, the first things you should consider is what pages you would need in the CMS.

There are 3 page types: Record, Push Notification, and User Management. You can learn about what each type does in [CMS overview]().

To show you how to configure the pages, let's assume we have a database like this:

|Record Type| Fields| ||||
|---|---|---|---|---|---|
|blogpost|_id|_created_at|title|content|cover|
|comment|_id|_created_at|blogpostId|

Suppose we want to show all the blogpost records and all the comment records in the CMS, and also need the push notification dashboard and the user management dashboard:

```yml
site:
  - type: Record
    # this is the CMS record name, which can be configured below
    name: blogpost
    label: Blogposts
  - type: Record
    name: comment
    label: Comments
  - type: PushNotifications
    label: Push
  - type: UserManagement
    label: User management

records:
  blogpost:
    # this is the CMS record name, you can call it anything
    record_type: blogpost
    # this is the database table that is linked to this CMS record.
  comment:
    record_type: comment
```

 `name` refers to the CMS record name. You can configure the CMS record name in the records section. `label` refers to the name to be shown in the CMS. If you don't provide a label, Skygear CMS will simply show the CMS record name.

Now save the changes and open the CMS. You should see all the pages on the left side bar.

![CMS Site](/assets/cms/cms-site.png)

## Configuring record pages

A record page has 4 views: **list**, **show**, **edit** and **new**. They are the ways your business users read (list, show), update (edit) or create (new) a record in the CMS.

### List view

![CMS List View](/assets/cms/cms-list.png)

You can imagine the list view as a spreadsheet that lists out all the records you have in a table. To show you how to configure the record page list view in the CMS, let's assume we have a database like this:

|Record Type| Fields| ||||
|---|---|---|---|---|---|
|blogpost|_id|_created_at|title|content|cover|
|comment|_id|_created_at|blogpostId|

This is what we need:

1. Display only the **title**, the  **content**, the **cover** and the **created time** of a blogpost.
2. Blog posts to be filterable with keywords in **title**.
3. Display only blog posts that are created in 2018.
4. Blog posts sorted in descending order based on their created time.

Then we should write in the YML:

```yml
site:
  - type: Record
    name: blogpost
    label: Blogposts

records:
  blogpost:
    record_type: blogpost
    list:
      fields:
        - name: title
          type: String
          label: Title
        - name: content
          type: TextArea
          label: Content
        - name: cover
          type: ImageAsset
          label: Cover
        - type: _created_at
      filters:
        - name: title
          type: String
          label: Title
      predicates:
        - name: title
          predicate: EqualTo
          value: ""
      default_sort:
        name: _created_at
        ascending: false

```

Let's have a look at all the configurable items:

**1. record_type**

It refers to the database table the CMS record is linked to. In the above example, the CMS record `blogpost` is linked to the database table `blogpost`.

**2. fields**

It refers to the fields you want to display. You may not want to display all the fields of a record as it may confuse your business users. For example, most developers choose to not display the record id.

When you configure a field, you have to specify its field types. **Field types in the CMS is not equivalent to data type the database**. They are the way the CMS displays the content of the field. For example, as blog posts are usually long texts, using TextArea to display the `content` field maybe more appropriate. You can learn about all the field types in [Explaining fields types]().

**3. filter**

It refers to the fields your business users can use to filter the records. For example, if you set the `title` field to be filterable, then the business users can filter blog posts that contains a specific keyword in its title.

**4. predicates**

Different from filter, predicates will be applied to the list view automatically. In the example, we have applied a predicate to the blogpost CMS record to show only blog posts created in 2018.

Below are the predicates you can use:
```
Like
NotLike
CaseInsensitiveLike
CaseInsensitiveNotLike
EqualTo
NotEqualTo
GreaterThan
GreaterThanOrEqualTo
LessThan
LessThanOrEqualTo
Contains
NotContains
ContainsValue
NotContainsValue
```
**5. default sort**

It refers to the field you want to use to order your records.

### Show view

![CMS show](/assets/cms/cms-show.png)

The show view is actually the detail page of a record. You can enter the show view by clicking the `show` button on the right hand side of a record in the list view.

You can configure what fields to be displayed in the show view. Following our sample database, suppose we want to show the **id**, the **title**, the **content**, the **image cover** and the **created date** of a blog post:

```yml
site:
  - type: Record
    name: blogpost
    label: Blogposts

records:
  blogpost:
    record_type: blogpost
    list:
      fields:
        - name: title
          type: String
          label: Title
        - name: content
          type: TextArea
          label: Content
        - name: cover
          type: ImageAsset
          label: Cover
        - type: _created_at
      filters:
        - name: title
          type: String
          label: Title
      default_sort:
        name: _created_at
        ascending: false
    show:
      fields:
        - type: _id
        - name: title
          type: String
          label: Title
        - name: content
          type: TextArea
          label: Content
        - name: cover
          type: ImageAsset
          label: Cover
        - type: _created_at
```

### Edit & New view

![CMS edit](/assets/cms/cms-edit.png)
![CMS new](/assets/cms/cms-new.png)

As suggested by their names, the edit view refers to the **edit page** where users can update a record; the new view refers to the **create page** where users can create a record. As the configuration of the two are very similar, the example below will show you how to configure the two in one go.

Similar to the show view, you can configure what fields to be displayed in the edit view and the new view.

Suppose business users can update the **title**, the **content** and the **cover image** of the blog posts, and when they create a new blog post, they can also enter its **title**,  its **content** and its **cover image**:

```yml
site:
  - type: Record
    name: blogpost
    label: Blogposts

records:
  blogpost:
    record_type: blogpost
    list:
      fields:
        - name: title
          type: String
          label: Title
        - name: content
          type: TextArea
          label: Content
        - name: cover
          type: ImageAsset
          label: Cover
        - type: _created_at
      filters:
        - name: title
          type: String
          label: Title
      default_sort:
        name: _created_at
        ascending: false
    show:
      fields:
        - type: _id
        - name: title
          type: String
          label: Title
        - name: content
          type: TextArea
          label: Content
        - name: cover
          type: ImageAsset
          label: Cover
        - type: _created_at
    edit:
      fields:
        - name: title
          type: String
          label: Title
        - name: content
          type: WYSIWYG
          label: Content
        - name: cover
          type: ImageAsset
          label: Cover
    new:
      fields:
        - name: title
          type: String
          label: Title
        - name: content
          type: WYSIWYG
          label: Content
        - name: cover
          type: ImageAsset
          label: Cover
```

Here we have used WYSIWYG for the content's field type in the edit and the new view. WYSIWYG is another field type the CMS support. It offers a WYSIWYG editor for business users to style the content. This is to illustrate that you can have different field types in different views to serve different purposes.

## Explaining record types in the CMS

In our above examples, whenever you need to configure a field, you have to specify **its name**, **its label** and **its field type**. Since **field types in the CMS is not equivalent to data type at database level**, in this section we will go into more details about the field types in CMS.

Fields types in the CMS refers to the way the CMS display the data. For example, suppose there are two roles in your app: student and teacher. While in the database role has a data type of **String**, however, in the CMS, as you may want to limit the user input to student or teacher, you can set the role's field type to **Dropdown** in the CMS.

```yml
records:
  user:
    edit:
      fields:
        - name: role
          label: role
          type: Dropdown
          options:
            - label: student
              value: student
            - label: teacher
              value: teacher
```

Here's a list of available types:

* Skygear reserved columns (see **_created_at** in the above example. more about Skygear reserved columns [here](https://docs.skygear.io/guides/advanced/database-schema/#records-table). )
* String = short texts
* TextArea = long texts
* Dropdown = dropdown boxes
* WYSIWYG = WYSIWYG editor
* DateTime = date time picker
* Boolean = boolean
* Integer = integer
* Number = nubmer
* ImageAsset = image
* FileAsset = files
* Reference = reference

Here is a full example of using all the field types.

```YML
fields:
  - type: _id
  - type: _created_at
  - type: _updated_at
  - name: name
    label: Name
    type: String
  - name: textarea
    type: TextArea
  - name: dropdown
    type: Dropdown
    null:
      enabled: false
      label: Undefined
    custom:
      enabled: true
    options:
      - label: Option A
        value: A
      - label: Option B
        value: B
      - label: Option S
        value: S
  - name: wysiwyg
    type: WYSIWYG
  - name: datetime
    type: DateTime
  - name: boolean
    type: Boolean
  - name: integer
    type: Integer
  - name: number
    type: Number
  - name: reference
    type: Reference
    reference_target: ref_demo
    reference_field_name: name
  - name: back_refs
    type: Reference
    reference_via_back_reference: back_ref_demo
    reference_from_field: reference
    reference_field_name: name
  - name: imageasset
    type: ImageAsset
  - name: fileasset
    type: FileAsset
```

While most types are pretty self explanatory itself, we will go into more details about the reference type in the next section.

## Working with reference in the CMS

Very often there is relation records (Skygear reference) in your app.

To show you how to reference works in the CMS, let's assume we have a database like this:

|Record Type| Fields| ||||
|---|---|---|---|---|---|
|blogpost|_id|_created_at|title|content|cover|
|comment|_id|_created_at|blogpostId|

The **comment** record has a reference field (blogpostId) that points to the blogpost record. Suppose we want to show all the comments a blogpost has in the blogpost's list and show view:

```yml
site:
  - type: Record
    name: blogpost
    label: Blogposts

records:
  comment:
    record_type: comment
  blogpost:
    record_type: blogpost
    list:
      fields:
        - name: title
          type: String
          label: Title
        - name: comment
          type: Reference
          reference_via_back_reference: comment
          reference_from_field: blogpostId
          reference_field_name: comment
    show:
      fields:
        - name: comment
          type: Reference
          reference_via_back_reference: comment
          reference_from_field: blogpostId
          reference_field_name: comment
```
As the CMS needs to display the comments, we need to create a CMS record 'comment' that links to the comment table in the database, even if comment doesn't take up a page in the CMS.

Explaining the configurable items:

1. **reference_via_back_reference**: the back referenced CMS record, in this case 'comment'
2. **reference_from_field**: the field that links up the two records, in this case 'blogpostId'
3. **reference_field_name**: the field we want to display from a comment record, in this case 'comment'

At the same time, we also want to show the blog post the comment belongs to in the comment's list and show view:

```yml
site:
  - type: Record
    name: comment
    label: Comments

records:
  blogpost:
    record_type: blogpost
  comment:
    record_type: comment
    list:
      fields:
        - name: comment
          type: String
          label: Comment
        - name: blogpostId
          type: Reference
          reference_target: blogpost
          reference_field_name: title
    show:
      fields:
        - name: comment
          type: String
          label: Comment
        - name: blogpostId
          type: Reference
          reference_target: blogpost
          reference_field_name: title
```
Explaining the configurable items:

1. **reference_taget**: the referenced record name, in this case 'blogpost'
2. **reference_field_name**: the field we want to display from the blogpost record, in this case 'title'.

## Configuring the push notification page

![CMS push](/assets/cms/cms-push.png)

To add a push notification dashboard in your CMS, not only do you need to add a `PushNotications` page under the site configuration, you also need to enable it in the `push_notifications` configuration.

```yml
site:
  - type: PushNotifications
    label: Push

push_notification:
  enabled: true
```

## Configuring the user management page

![CMS user](/assets/cms/cms-user.png)

Similar to adding a push notification dashboard in the CMS, to add a user management dashboard, not only do you need to add a `UserManagement` page under the site configuration, you also need to enable it in the `user_management` configuration.

```yml
site:
  - type: UserManagement
    label: User Management

user_management:
  enabled: true
```
