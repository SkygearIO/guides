---
title: Configuring the CMS Record page
---
[[toc]]

In [CMS basics][doc-cms-basic] we learn that the CMS is consisted of 3 types of pages: Records, User management and Push notification. In this section, we will go into details of the record page configuration.

A record page has 4 views: **list**, **show**, **edit** and **new**. They are the ways your business users read (list, show), update (edit) or create (new) a record in the CMS.

## List view

![Skygear CMS List View](/assets/cms/cms-list.png)

You can imagine the list view as a spreadsheet that lists out all the records you have in a table. 

Example usage: 

Assume we have the following tables in our app.

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

1. Display only the **title**, the  **content**, the **cover** and the **created time** of a blogpost.
2. Show the comments a blog post has.
3. Users can filter blog posts with **title** keywords.
4. Display only blog posts that are created in 2018.
5. Blog posts sorted in descending order based on their created time.

Then we should write in the YML:

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
        - name: content
          type: TextArea
          label: Content
        - name: cover
          type: Image
          label: Cover
        - name: _created_at
        - name: blogpost_comment
          label: Comments
          type: ReferenceList
          reference_via_back_reference: comment
          reference_from_field: blogpostId
          reference_field_name: comment
      filters:
        - name: title
          type: String
          label: Title
      predicates:
        - name: _created_at
          predicate: GreaterThanOrEqualTo
          value: 2018-01-01
      default_sort:
        name: _created_at
        ascending: false
```

### List view configurable items

- [`record_type`](./#record_type)
- [`fields`](./#fields)
- [`filters`](./#filters)
- [`predicates`](./#predicates)
- [`default_sort`](./#default_sort)
- [`actions`](./#actions)
- [`item_actions`](./#item_actions)

<a name='record_type'></a>
#### `record_type`

```yml
record_type: database table name
```

It refers to the database table the CMS record is linked to. In the above example, the CMS record `blogpost` is linked to the database table `blogpost`.

<a name='fields'></a>

#### `fields`

```yml
fields:
  - name: field name in the database
    type: CMS record type # see the field type guide
    label: name to display in the CMS
```

It refers to the fields you want to display. You may not want to display all the fields of a record as it may confuse your business users. For example, most developers choose to not display the record id.

There are three reserved fields you can use:

```yml
fields:
  - name: _id
  - name: _created_at
  - name: _updated_at
```

Besides, when you configure a field, you have to specify its field types. **Field types in the CMS is not equivalent to data type the database**. They are the way the CMS displays the content of the field. For example, as blog posts are usually long texts, using TextArea to display the `content` field maybe more appropriate. You can learn about all the field types in the [CMS field type guide][doc-cms-field].


<a name='filters'></a>

#### `filters`

```yml
filters:
  - name: field name in the database
    type: filter type 
    label: name to display in the CMS
```

It refers to the fields your business users can use to filter the records. For example, if you set the `title` field to be filterable, then the business users can filter blog posts that contains a specific keyword in its title.

`filter type` determines the filter format. For example, if the filter type is `String`, a keyword search filter will be used.

Below are the filter type you can use:

- `String`
- `DateTime`
- `Boolean`
- `Integer`
- `General`
- `Reference`

You can have multiple filters. All filters can be accessed by pressing the 'filter' button on the upper right corner.

<a name='predicates'></a>

#### `predicates`
```yml
predicates:
  - name: database field name
    predicate: predicate
    value: value the predicate based on
```
Different from filter, predicates will be applied to the list view automatically. For example, if we only want to show blog posts created in 2018, we can apply a predicate to the `_created_at` field.

Below are the predicates you can use:

- `Like`
- `NotLike`
- `CaseInsensitiveLike`
- `CaseInsensitiveNotLike`
- `EqualTo`
- `NotEqualTo`
- `GreaterThan`
- `GreaterThanOrEqualTo`
- `LessThan`
- `LessThanOrEqualTo`
- `Contains`
- `NotContains`
- `ContainsValue`
- `NotContainsValue`

Example usage:

Suppose we want to display blogpost that are created in 2018 only.

```yml
list:
  predicates:
    - name: _created_at
      predicate: GreaterThan
      value: 2018-01-01 
```
Note: for date value, we support full date or datetime. For example, 2018-01-01 or 2018-01-01 12:0:0.

<a name='default_sort'></a>

#### `default_sort`
```yml
default_sort:
  name: database field name
  ascending: true or false
```
It refers to the fields you want to use to order your records.

<a name='actions'></a>

#### `actions`

![Skygear actions](/assets/cms/cms-actions.png)


```yml
actions:
  - type: AddButton
  - type: Import    
    name: name configured in the import section
    label: Text to show inside the button
    atomic: true or false (optional)
  - type: Export
    name: name configured in the export section
    label: Text to show inside the button
  - type: Link
    label: Text to show inside the button
    href: link path
    target: HTML link target attribute
```

By default, there will be two buttons on the upper right hand corner: 'Add' and 'Add Filter'. On top of that, you can create more action buttons by configuring `actions`.

There are 4 action types:

- `Import`: import data with a CSV
- `Export`: import data to a CSV
- `Link`: any custom link
- `AddButton`: create a new record

If you want to use the import and export function, you also need to configure the import or the export settings. Read the 'CMS Import and Export' guide to learn more.

<a name='item_actions'></a>

#### `item_actions`
![Skygear item actions](/assets/cms/cms-item-actions.png)

```yml
item_actions: 
  - type: ShowButton
  - type: EditButton
  - type: Link
    label: Text to show inside the button
    href: link path
    target: HTML link target attribute
```
There are buttons next to every record, and they are called 'item actions'. If you have enabled the edit and the show view, by default there are two buttons next to each record: show and edit.

To add custom buttons, simply configure `item_actions`. For example, you may add a button next to each record that links users to the Google search results of the record's title.

```yml
  - type: Link
    label: Search
    href: https://www.google.com/search?q={record.title}
    target: _blank
```

## Show, edit and new view
<em>Show view</em>
![CMS show](/assets/cms/cms-show.png)

<em>Edit view</em>
![CMS edit](/assets/cms/cms-edit.png)

<em>New view</em>
![CMS new](/assets/cms/cms-new.png)

As suggested by their names:
1.  **show** refers to the **detail page** where users can see the details of an individual record
2. **edit** refers to the **edit page** where users can update a record
3. **new** refers to the **create page** where users can create a record. 

Since [`fields`](./#fields) is the only configurable item of the 3 views, the below example will demonstrate how to configure them in one go. 

Note: you can have different CMS field type for the same field for different views. For example, you may use `TextArea` to display the blog post's content in the **list** view and WYSIWYG in the **new** view. In the [CMS field type guide][doc-cms-field], you will see all the available field types and their respective UI views.

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
        - name: content
          type: TextArea
          label: Content
        - name: cover
          type: Image
          label: Cover
        - name: _created_at
        - name: blogpost_comment
          label: Comments
          type: ReferenceList
          reference_via_back_reference: comment
          reference_from_field: blogpostId
          reference_field_name: comment
      actions:
        - type: AddButton
        - type: Link
          label: Custom button
          href: https://skygear.io
          target: _blank
      item_actions:
        - type: ShowButton
        - type: EditButton
        - type: Link
          label: Custom button
          href: https://skygear.io
      filters:
        - name: title
          type: String
          label: Title
      predicates:
        - name: _created_at
          predicate: GreaterThanOrEqualTo
          value: 2018-01-01
      default_sort:
        name: _created_at
        ascending: false
    show:
      fields:
        - name: title
          type: String
          label: Title
        - name: content
          type: TextArea
          label: Content
        - name: cover
          type: Image
          label: Cover
        - name: _created_at
        - name: blogpost_comment
          label: Comments
          type: ReferenceList
          reference_via_back_reference: comment
          reference_from_field: blogpostId
          reference_field_name: comment
    edit:
      fields:
        - name: title
          type: String
          label: Title
        - name: content
          type: WYSIWYG
          label: Content
        - name: cover
          type: Image
          label: Cover
        - name: _created_at
        - name: blogpost_comment
          label: Comments
          type: ReferenceList
          reference_via_back_reference: comment
          reference_from_field: blogpostId
          reference_field_name: comment
    new: 
      fields:
        - name: title
          type: String
          label: Title
        - name: content
          type: WYSIWYG
          label: Content
        - name: cover
          type: Image
          label: Cover
        - name: _created_at
        - name: blogpost_comment
          label: Comments
          type: ReferenceList
          reference_via_back_reference: comment
          reference_from_field: blogpostId
          reference_field_name: comment
```

If the 4 views share the same fields config, you can use anchors to save time.

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
      fields: &blogpost_fields
        - name: title
          type: String
          label: Title
        - name: content
          type: TextArea
          label: Content
        - name: cover
          type: Image
          label: Cover
        - name: _created_at
        - name: blogpost_comment
          label: Comments
          type: ReferenceList
          reference_via_back_reference: comment
          reference_from_field: blogpostId
          reference_field_name: comment
      actions:
        - type: AddButton
        - type: Link
          label: Custom button
          href: https://skygear.io
          target: _blank
      item_actions:
        - type: ShowButton
        - type: EditButton
        - type: Link
          label: Custom button
          href: https://skygear.io
      filters:
        - name: title
          type: String
          label: Title
      predicates:
        - name: _created_at
          predicate: GreaterThanOrEqualTo
          value: 2018-01-01
      default_sort:
        name: _created_at
        ascending: false
    show:
      fields: *blogpost_fields
    edit:
      fields: *blogpost_fields
    new: 
      fields: *blogpost_fields
```

## What's next?

Next, check out the [field types supported by the Skygear CMS][doc-cms-field].

[doc-cms-basic]: /guides/cms/cms-basics/
[doc-cms-field]: /guides/cms/cms-field-type/