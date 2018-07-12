---
title: Configuring Imports and Exports
---

[[toc]]

Handling bulk data in Skygear CMS is easy. Your users can import data with a CSV and export data to a CSV. 

## Imports

![Skygear CMS import](/assets/cms/cms-import.png)

Your users can import data to Skygear with a CSV. To do so, you need to configure the import settings and add an import button to the right page.

### Configuring imports

There are a few items you can configure in the import settings, and we are using the following 2 tables to illustrate how.

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
|comment| String|
|blogpostId | Reference|

This is what we need:

1. Allow users to import a list of comments. 
2. We will use `_id` as an identifier. If there is a comment in the database with the same `_id`, we will update the record. 
3. However,if there are more than 1 comment with the same `_id` in the database, we will throw an error to the users.
4. To correctly associate every comment to a blogpost, we should also input the blogpost's title next to each comments.

```yml
site:
  - type: Record
    name: comment
  - type: Record
    name: blogpost

imports:
  import-comment:
    record_type: comment
    identifier: content
    handle_duplicated_identifier: throw-error
    fields:
      - name: comment
      - name: blogpostId
        reference_target: blogpost
        reference_field_name: title
        handle_duplicated_reference: use-first

records:
  comment: 
    record_type: comment
  blogpost: 
    record_type: blogpost
```

This is what your CSV should look like:

|_id|comment| title|
|--- |---| ---|
|0001-20180909|Can't agree more| Adele is so adorable!|
|0002-20180910|Let's go drinking this Friday! |Today is such a tired day|
|0003-20180911|Of course! |Skygear is so awesome|

Configurable items:

- <a href='./#record_type'>`record_type`</a>
- <a href='./#identifier'>`identifier`</a>
- <a href='./#handle_duplicated_identifier'>`handle_duplicated_identifier`</a>
- <a href='./#fields'>`fields`</a>

<a name='record_type'></a>

#### `record_type`
The database table this import is linked to.
```yml
imports:
  import-name-random: 
    record_type: 
```

<a name='identifier'></a>

#### `identifier`
The field that Skygear uses to identify a record. If `identifier` is not provided, Skygear will use `_id` to identify records. 

If a record in the database is found to have the same identifier as the imported record, Skygear will update the record in the database.

```yml
imports:
  import-name-random: 
    identifier: 
```
<a name='handle_duplicated_identifier'></a>

#### `handle_duplicated_identifier`
The way to handle duplicated records when If more than 1 record in the database is found to have the same identifier as the imported record.

```yml
imports:
  import-name-random: 
    handle_duplicated_identifier: throw-error or use-first
```

- `throw-error` will return an error about this record. If you want to reject the whole import when there are errors, you can set `atomic: true` in the `action` setting (will be explained below).

- `use-first` will randomly accept one of the duplicated records.

<a name='fields'></a>

#### `fields`
Fields the users should fill in. 

For relational records, Import support only **reference** (i.e. 1-many).

```yml
imports:
  import-name-random: 
    fields:
      - name: a normal field
      - name: a reference field
        reference_target: the referenced table 
        reference_field_name: the field user should fill in the CSV to idenitfy the referenced record
        handle_duplicated_reference: throw-error or use-first
```
- `throw-error` will return an error. If you want to reject the whole import when there are errors, you can set `atomic: true` in the `action` setting (will be explained below).

- `use-first` will randomly assign the record to one of the duplicated references.

### Add the import button

To add an import button to the page, you need configure it in the list view. 

Example usage:

Suppose now we are adding an **import** button on the `comment`'s list view.

```yml
site:
  - type: Record
    name: comment

records:
  comment:
    record_type: comment
    list:
      actions:
        - type: Import
          name: import-comment # this is defined in the import section
          label: Import
          atomic: true 
```

- `atomic`: When `atomtic` is set to true, the whole import will be rejected if there are errors from `handle_duplicated_reference` and `handle_duplicated_identifier`. Default is false.

## Exports

![Skygear CMS export](/assets/cms/cms-export.png)

Your users can export data from Skygear to a CSV. To do so, you need to configure the export settings and add an export button to the right page.

### Configuring exports

Reusing the two databases in the Configuring imports section as an example.

This is what we need:

1. Allow users to export all the comments
2. Next to each comment, we will also show the blogpost title the comment belongs to

```yml
site:
  - type: Record
    name: comment
    label: Comment

records:
  comment: 
    record_type: comment
  blogpost:
    record_type: blogpost

exports:
  export-comments:
    record_type: comment
    fields:
      - name: _id
      - name: _created_at
      - name: comment
      - name: blogpost_title
        reference_target: blogpost
        reference_field_name: title
```
The exported CSV will look like this:

|_id|_created_at| comment| title|
|--- |---|---| ---|
|93ab4e95-2cb6-4c00-890e-20038d1be03c|2018-07-09 18:00:56+08:00|Can't agree more| Adele is so adorable!|
|8cbb14e7-0cb0-48c8-a46e-ba1cfb8804f6|2018-07-09 17:59:52+08:00|Getting a drink. Anyone? |Today is such a tired day|
|90534306-48ff-4b38-8b01-6fa53831096b|2018-07-09 17:59:21+08:00| Of course! |Skygear is so awesome|

Export supports also back reference (i.e. many-1). The example below will show you how to configure back reference.

This is what we need:
1. Allow users to export all the blog posts
2. Next to each blogpost, we will also show the comments belong to this blogpost

```yml
site:
  - type: Record
    name: blogpost
    label: Blogpost

records:
  comment: 
    record_type: comment
  blogpost:
    record_type: blogpost

exports:
  export-blogposts:
    record_type: blogpost
    fields:
      - name: _id
      - name: _created_at
      - name: content
      - name: title
      - name: comment_comment
        reference_via_back_reference: comment
        reference_from_field: blogpostId
        reference_fields:
          - name: _id
            label: comment_id
          - name: comment
            label: comment
```
The exported CSV will look like this:

|_id|_created_at| title| content| comment_id| comment|
|--- |---|---| ---| ---| ---|
|93ab4e95-2cb6-4c00-890e-20038d1be03c|2018-07-09 18:00:56+08:00|What do you think about the new CMS?|Let us know what you think!| 9opb4e15-2f03-9f00-890e-20037w1bg03c, 6f4cc50c-ac72-45a6-8308-d54ce69a0a3b| Better than the old one., Awesome!|
|8cbb14e7-0cb0-48c8-a46e-ba1cfb8804f6|2018-07-09 17:59:52+08:00|Let's go drinking this Friday! |Today is such a tired day|43c48ee9-558b-43a4-ba85-7e6e3cdd38bd|Let's go drinking this Friday!|
|90534306-48ff-4b38-8b01-6fa53831096b|2018-07-09 17:59:21+08:00|Skygear is so awesome | Don't you think so? | 9196132f-4507-44fd-aadc-e60d76f9ec6d| Of course!|


### Add the export button
Now, let's add an **export** button to the `comment`'s list view.
```yml
site:
  - type: Record
    name: comment

records:
  comment:
    record_type: comment
    list:
      actions:
        - type: Export
          name: export-comments # this is defined in the import section
          label: Export
```