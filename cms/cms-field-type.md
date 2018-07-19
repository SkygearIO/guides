---
title: CMS field types
---

[[toc]]

## Introduction

When you configure `fields` of a CMS record, you need to define the type of each field.

```yml
records:
  blogpost:
    record_type: blogpost
      list:
        fields:
          - name: content
            type: string # CMS field types
            label: Content
```

### Field type =/= Data type

Field type in Skygear CMS refers to the UI presentation of the field, not its data type in the database.

For example, suppose there are two roles in your app: student and teacher. In the user table, there is a column named 'role' the stores the role of the user. 'role' in the database has a data type of `string`, however, in the CMS, you can set the its field type to `dropdown` to limit the input to student or teacher.

```yml
records:
  user:
   record_type: user
     list:
       fields:
          - name: role
            type: dropdown # CMS field types
            label: Role
            options:
              - label: Teacher
                value: teacher
              - label: Student
                value: student
```

### You can use different field type for the same field in different views

A CMS record has 4 views: list, show, edit and new. It is likely that the 4 views have similar fields. However, considering the UI presentation of the 4 views are different, you may want to use different field type in different view. 

Reusing the teach and student example. Suppose you want to display the role as `string`, however, when users edit or create a new user, you want to display the role field as `dropdown` so that you can limit the user input to student or teacher. 

``` yml
records:
  blogpost:
    record_type: user
      list:
        fields:
          - name: role
            type: string
            label: Role
      show:
        fields:
          - name: role
            type: string
            label: Role      
      edit:
        fields:
          - name: role
            type: dropdown
            label: Role
      new:
        fields:
          - name: role
            type: dropdown
            label: Role      
```

## Supported field types

Every field type has two views: display and edit. Display view default will be applied to the list and the show view, while edit view default will be applied to the edit and new view.

For example, if you use `wysiwyg` in the list or the show view, you will get the default WYSIWYG display view (a HTML preview WYSIWYG widget). If you use `wysiwyg` in the edit or new view, you will get the default WYSIWYG edit view (a working WYSIWYG editor).

<table>
  <tr>
    <td><strong>Category</strong></td>
    <td><strong>Field type</strong></td>
    <td><strong>Display view</strong></td>
    <td><strong>Edit view</strong></td>
  </tr>
  <tr>
    <td rowspan='4'>Text</td>
    <td><a href='./#string'><code>string</code></a></td>
    <td>Single-line text</td>
    <td>Single-line text input</td>
  </tr>
  <tr>
    <td><a href='./#textarea'><code>text_area</code></a></td>
    <td>Mulitple-line text</td>
    <td>Mulitple-line text input</td>
  </tr>
  <tr>
    <td><a href='./#wysiwyg'><code>wysiwyg</code></a></td>
    <td>HTML preview WYSIWYG widget</td>
    <td>WYSIWYG Eidtor</td>
  </tr>
  <tr>
    <td><a href='./#dropdown'><code>dropdown</code></a></td>
    <td>Non-expandable dropdown</td>
    <td>Dropdown input</td>
  </tr>
  <tr>
    <td rowspan='3'>Number</td>
    <td><a href='./#number'><code>number</code></a></td>
    <td>Number display</td>
    <td>Number input</td>
  </tr>
  <tr>
    <td><a href='./#float'><code>float</code></a></td>
    <td>Float display</td>
    <td>Float input</td>
  </tr>
  <tr>
    <td><a href='./#integer'><code>integer</code></a></td>
    <td>Integer display</td>
    <td>Integer input</td>
  </tr>
  <tr>
    <td rowspan='2'>Files</td>
    <td><a href='./#asset'><code>asset</code></a></td>
    <td>Asset name</td>
    <td>Asset uploader</td>
  </tr>
  <tr>
    <td><a href='./#image'><code>image</code></a></td>
    <td>Image display</td>
    <td>Image uploader</td>
  </tr>
  <tr>
    <td rowspan='4'>Relational records</td>
    <td><a href='./#reference'><code>reference</code></a> (1-many)</td>
    <td>Text display</td>
    <td>Dropdown input</td>
  </tr>
  <tr>
    <td><a href='./#referencelist'><code>reference_list</code></a> (many-1 or many-many)</td>
    <td>Text display</td>
    <td>Dropdown input</td>
  </tr>
  <tr>
    <td><code>embedded_reference</code></td>
    <td colspan='2'>Coming soon</td>
  </tr>
  <tr>
    <td><code>embedded_reference_list</code></td>
    <td colspan='2'>Coming soon</td>
  </tr>
  <tr>
    <td rowspan='2'>Time</td>
    <td><a href='./#datetime'><code>date_time</code></a></td>
    <td>Date time display</td>
    <td>Date and time picker</td>
  </tr>
  <tr>
    <td><code>date</code></td>
    <td colspan='2'>Coming soon</td>
  </tr>
  <tr>
    <td>Boolean</td>
    <td><a href='./#boolean'><code>boolean</code></a></td>
    <td>Non-editable switch</td>
    <td>Switch</td>
  </tr>  
  <tr>
    <td rowspan='2'>Location</td>
    <td><code>location</code></td>
    <td colspan='2'>Coming soon</td>
  </tr>
  <tr>
    <td><code>map</code></td>
    <td colspan='2'>Coming soon</td>
  </tr>
  <tr>
    <td>JSON</td>
    <td><code>json</code></td>
    <td colspan='2'>Coming soon</td>
  </tr>
</table>

<a name='string'></a>

#### `string`

Single-line text.

``` yml
fields:
  - name: database field name
    type: string
    default_value: default value # applicable only to the new view
    editable: true or false # applicable only to the edit and the new view
```

<a name='textarea'></a>

#### `text_area`

Multiple-line text

```yml
fields:
  - name: database field name
    type: text_area
    default_value: default value # applicable only to the new view
    editable: true or false # applicable only to the edit and the new view
```

<a name='wysiwyg'></a>

#### `wysiwyg`

WYSIWYG with HTML markups. If you use `wysiwyg` in the edit and the new view, we also suggest you using `wysiwyg` in the list and the show view so that your users can see the HTML preview instead of raw HTML.

```yml
fields:
  - name: database field name
    type: wysiwyg
    config: tiny wysiwyg config
    default_value: default value # applicable only to the new view
    editable: true or false # applicable only to the edit and the new view

```
- `config`: Skygear CMS uses WYSIWYG editor from [Tinymce](https://www.tinymce.com/). Therefore, you can configure the WYSIWYG editor according to the [Tinymce doc](https://www.tinymce.com/docs/configure/). Below is the configuration Skygear CMS is currently using.

    ```javascript
    {
      height: 480,
      plugins:
        'anchor charmap code colorpicker contextmenu fullscreen hr image imagetools link lists nonbreaking paste tabfocus table textcolor',
      skin_url: 'tinymce/skins/lightgray',
      toolbar1:
        'bold italic strikethrough forecolor backcolor | link image | alignleft aligncenter alignright alignjustify  | numlist bullist outdent indent  | removeformat',
    }
    ```

Example usage:

Suppose we want to add another toolbar to the editor:

```yml
fields:
  - name: database field name
    type: wysiwyg
    config:
      toolbar2: paste | copy | undo | redo
```

<a name='dropdown'></a>

#### `dropdown`

Currently `dropdown` only support `string` inputs.

```yml
fields:
  - name: database field name
    type: dropdown
    custom: 
      enabled: true or false
      label: name of the custom value
    null: 
      enabled: true or false
      label: name of the null values
    options:
      - label: option 1
        value: A # only string inputs are supported
      - label: option 2
        value: B
    default_value: default value # applicable only to the new view
    editable: true or false # applicable only to the edit and the new view
```

- `custom`: When custom is enabled, users will be given an 'other' option and be able to enter custom value. Default is false.

- `null`: When null is enabled, users can submit a null value. You can also give a display name to the null value (i.e. undefined). Default is false.

- `options`: All available options.


<a name='number'></a>

#### `number`

```yml
fields:
  - name: database field name
    type: number
    default_value: default value # applicable only to the new view
    editable: true or false # applicable only to the edit and the new view    
```

<a name='float'></a>

#### `float`

```yml
fields:
  - name: database field name
    type: float
    default_value: default value # applicable only to the new view
    editable: true or false # applicable only to the edit and the new view
```

<a name='integer'></a>

#### `integer`

```yml
fields:
  - name: database field name
    type: integer
    default_value: default value # applicable only to the new view
    editable: true or false # applicable only to the edit and the new view
```
<a name='asset'></a>

#### `asset`

Files except images.

```yml
fields:
  - name: database field name
    type: asset
    accept: the accepted file types
    nullable: 
      enabled: true or false
    editable: true or false # applicable only to the edit and the new view
```

- `nullable`: `nullable` only applies to the edit and the new view. When `nullable` is enabled, users can update or create a record even if the asset field is null. When `nullable` is disabled, users must upload an asset when they edit or create a record.
- `type`: Read the MDN web doc [here](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#attr-accept) to learn more about the file types.

<a name='image'></a>

#### `image`

```yml
fields:
  - name: database field name
    type: image
    nullable: 
      enabled: true or false
    editable: true or false # applicable only to the edit and the new view
```
- `nullable`: `nullable` only applies to the edit and the new view. When `nullable` is enabled, users can update or create a record even if the image field is null. When `nullable` is disabled, users must upload an image when they edit or create a record.

<a name='reference'></a>

#### `reference`

1-to-1 relational records. 

```yml
fields:
  - name: database field name
    type: reference
    reference_target: Referenced CMS record
    reference_field_name: Referenced record field name
    editable: true or false # applicable only to the edit and the new view
```

Example usage:

Suppose we have a `comment` table with every comment associated to a blogpost, and you want to show the blogpost's title next to the comments.

```yml
site:
  type: record
  name: comment
  label: Comment

records:
  blogpost: 
    record_type: blogpost
  comment:
    record_type: comment
    list: 
      fields: &blogpost_reference_field
        - name: blogpostId
          type: reference
          reference_target: blogpost
          reference_field_name: title
    show: 
      fields: *blogpost_reference_field
    edit:
      fields: *blogpost_reference_field
    new:
      fields: *blogpost_reference_field
```

<a name='referencelist'></a>

#### `reference_list`

Many-to-1 and/or many-to-many relation records.

**many-to-1**

```yml
fields:
  - name: a random name
    type: reference_list
    reference_via_back_reference: the table the field refers to
    reference_from_field: the field that links the two tables
    reference_field_name: the field you want to display
    editable: true or false # applicable only to the edit and the new view
```

Example usage:

Suppose we have a `comment` table with every comment referenced to a blogpost, and you want to show all the comments a blogpost has.

```yml
site:
  type: record
  name: blogpost
  label: Blogpost

records:
  comment: 
    record_type: comment
  blogpost:
    record_type: blogpost
      list: 
        fields: &comment_reference_field
          - name: blogpost
            type: reference_list
            reference_via_back_reference: comment
            reference_from_field: blogpostId
            reference_field_name: comment
      show: 
        fields: *comment_reference_field
      edit:
        fields: *comment_reference_field
      new:
        fields: *comment_reference_field
```

**many-to-many**

```yml
fields:
  - name: random
    reference_via_association_record: association record name
    reference_target: desired table
    reference_field_name: desired fields
    editable: true or false # applicable only to the edit and the new view
```
Displaying a many-to-many relational records in Skygear CMS is a little tricky. We call the intermediate tables that connect two tables **association records**. You need to configure the association record before you can display a many-to-many relation record.

Example usage:

Suppose we have an intermediate table named `blogpost_comment` that connects the `blogpost` and `comment` table. We want to show all the comments a blogpost has.

```yml
site:
  - type: record
    name: blogpost
    label: blogpost

records:
  comment: 
    record_type: comment
  blogpost:
    record_type: blogpost
    list:
      fields:
        - name: random name
          type: reference_list
          reference_via_association_record: blogpost_comment
          reference_target: comment
          reference_field_name: content

association_record:
  blogpost_comment:
    - name: blogpostId
      type: reference
      reference_target: blogpost
    - name: commendId
      type: reference
      reference_target: comment
```


<a name='datetime'></a>

#### `date_time`

Date with time.

```yml
fields:
  - name: database field name
    type: date_time
    timezone: timezone
    default_value: default value # applicable only to the new view
    editable: true or false # applicable only to the edit and the new view
```
- `timezone`: Setting the timezone of a `Datetime` field will override the default timezone setting. The timezone list can be found [here](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List). Use the TZ convention (e.g. Africa/Abidjan).

<a name='boolean'></a>

#### `boolean`

Switch.

```yml
fields:
  - name: database field name
    type: boolean
    default_value: true or false # applicable only to the new view
    editable: true or false # applicable only to the edit and the new view
```

## What's next?

Next, let's learn about how to [configure import and export][doc-cms-import].

[doc-cms-import]: /guides/cms/cms-import/