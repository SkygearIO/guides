---
title: Skygear Command Line Interface
---

Skygear provides a default command-line interface `skycli` to interact with
Skygear's handlers and PostgreSQL backend to facilitate application development
and some common administration works.

## Getting started

To install `skycli`:

```bash
$ mkdir -p $GOPATH/src/github.com/SkygearIO
$ cd $GOPATH/src/github.com/SkygearIO
$ git clone https://github.com/SkygearIO/skycli
$ go get -u github.com/SkygearIO/skycli
```

Then configure the API key of your Skygear app so that it can call Skygear's
handler:

```bash
$ export SKYCLI_API_KEY=YOUR_API_KEY
$ export SKYCLI_ENDPOINT=YOUR_ENDPOINT
```

Let's verify it is working by getting a non-existed record:

```bash
$ skycli record get note/id _id
Error: record not found
```

API key can also be overridden on a per command basis by supplying the
`api_key` flag:

```bash
$ skycli --api_key ANOTHER_API_KEY record get note/id _id
Error: record not found
```

::: tips

**Tips**: Global options like _API key_, _access token_ and _endpoint
address_ should be persisted on a project-by-project basis (e.g. on
`./.skycli/config`) so that CLI user doesn't have to configure it on every new
shell.

:::

## Manage Database Schema

`schema` sub-commands help to add, rename and delete record fields -- the kind
of schema operations that cannot be achieved via regular record handlers.

`schema` sub-commands are provided as a convenient tool for simple schema
operations and do not intended to be a full-featured database migration tool.
If you need more sophisticated amendments on the database, remember you have
full access to the underlying PostgreSQL db, you should feel free to write db
script or use whatever tools like `psql` to fulfill your needs. Refer to
database structure guide for more information on our internal table
definitions.

Managing record schema requires Skygear Server to be placed in development mode
or using a master key.

### Fetch schema

#### Description

`skycli schema fetch` fetches the record schema from the server.

#### Synopsis

```bash
$ skycli schema fetch
```

#### Examples

```bash
$ skycli schema fetch
{
  "user": {
    "fields": [
      { "name": "firstname", "type": "string" }
    ]
  }
}
```

### Add new column

#### Description

`skycli schema add` adds a new column to a record type.

#### Synopsis

```bash
$ skycli schema add <record_type> <column_name> <column_def>
```

`column_def` means the type of field you wanted to create. The permitted
values are:

* `string`
* `number`
* `boolean`
* `json`
* `location`
* `datetime`
* `asset`
* `ref(record_type)`, where `record_type` is an existing record type

#### Examples

```bash
$ skycli schema add user lastname string
$ skycli schema fetch
{
  "user": {
    "fields": [
      { "name": "firstname", "type": "string" },
      { "name": "lastname", "type": "string" }
    ]
  }
}
```

### Rename existing column

#### Description

`skycli schema move` renames an existing column to another name.

#### Synopsis

```bash
$ skycli schema move <record_type> <column_name> <new_column_name>
```

#### Examples

```bash
$ skycli schema move user lastname othername
$ skycli schema fetch
{
  "user": {
    "fields": [
      { "name": "firstname", "type": "string" },
      { "name": "othername", "type": "string" }
    ]
  }
}
```

### Remove existing column

#### Description

`skycli schema remove` removes an existing column.

#### Synopsis

```bash
$ skycli schema remove <record_type> <column_name>
```

#### Examples

```bash
$ skycli schema remove user othername
$ skycli schema fetch
{
  "user": {
    "fields": [
      { "name": "firstname", "type": "string" }
    ]
  }
}
```

## Manage Skygear Records

### Import

#### Description
`skycli record import` imports record data from JSON file.

When importing records, each record found in the file is saved to the
database. If the record to be saved already exists in the database, each
field in the record to be saved will overwrite the respective field in the
existing record.

Existing records and existing fields are not removed before the import
operation. Therefore, records and fields that exist in the database but
not in the file will remain in place.

#### Synopsis

```bash
skycli record import [options] [<path> ...]
```

Records data stored in `<path>` will be imported.<br>
If `<path>` is a directory, all files in the directory with `.json` extension are imported.<br>
If `<path>` is not specified, skycli will import from stdin.

For the file format of imported files, see [File format](#File format)


Use `skycli record import --help` to view a list of available options.


#### File Format <a name="File format"> </a>

Each JSON file/stdin follows the [concatenated JSON format](https://en.wikipedia.org/wiki/JSON_Streaming#Concatenated_JSON), where each record is represented by one JSON object.

##### Example
`records.json`:
```javascript
{ "_id": "student/44ea2a9c-ee15-447a-a758-430210be7fa4", "name": "Alice", "age": 10}
{ "_id": "city/f8cf1947-68ec-4fb0-9216-dc32ef92ddeb", "name": "Hong Kong", "image": "@file:images/hongkong.jpg" }
{
    "_id"       : "country/8b799ecd-e243-44e1-aa14-a736121defe1",
    "name"      : "Japan",
    "location"  : "@loc:3.14,2.15"
}
```

Each record is represented by a single JSON dictionary. The key `_id` is mandatory for every record, any other key that starts with underscore `_` is reserved by Skygear Server and should not be used.

#### Complex Value

Skycli supports a simpler format of complex values. For each value using this format, skycli will prompt the user for confirmation. Unconverted complex value will be stored literally as its simpler form. Use the flag `--no-warn-complex` to skip the prompt and convert automatically.

Skycli currently support the following complex values:

```
location      @loc:<lat>,<lng>
reference     @ref:<referenced_id>
string        @str:<literal>
```

See [Protocol Data Type](https://github.com/oursky/skygear/wiki/Protocol-DataType) for more complex value supported by Skygear.

#### Handling assets

For the field with value `@file:<relative_path>`, the corresponding asset file will be uploaded. When returning the field value from server to skycli, the field
will be replaced as `@asset:<asset_id>` to indicate an uploaded asset.

By default, the location of the asset file is relative to the imported JSON file. If `--basedir` is set, Skycli will find the asset file relative to `--basedir`.


#### Examples

##### Stdin:
```bash
$ skycli record import
{ "_id": "student/bed763f3-071f-4d87-91fb-dccb22099162", "name": "Alice", "age": 10}
{ "_id": "city/f8cf1947-68ec-4fb0-9216-dc32ef92ddeb", "name": "Hong Kong", "image": "@file:images/hongkong.jpg" }
{
    "_id"       : "country/805b13a2-22c2-4785-87bf-5a179413f0e8",
    "name"      : "Japan",
    "location"  : "@loc:3.14,2.15"
}
Found complex value @loc:3.14,2.15. Convert? (y or n) y
```

##### Files or Directories:
```bash
$ skycli record import city.json records --skip-asset
Found complex value @loc:3.14,2.15. Convert? (y or n) y
```

### Query

#### Description
`skycli record query` query records from Skygear.

All records with the specified record type will be fetched.

The result will be printed to stdout. If `-o` is specified, then the result will be stored with the given filename.

Each record will be printed as a JSON object delimited by a newline character. If `--pretty-print` is specified, then each record will be printed with proper indentation, otherwise each record will be printed in a single line.

For downloading assets, please see `skycli record get`

#### Synopsis

```bash
skycli record query [options] <record_type>
```

Use `skycli record query --help` to view a list of available options.

#### Examples

##### Record ID:
```bash
$ skycli record query city --pretty-print
{
    "_id": "city/f8cf1947-68ec-4fb0-9216-dc32ef92ddeb",
    "name": "hongkong"
}
{
    "_id": "city/f6890365-cca6-440d-a813-e2ace6662301",
    "country": "USA"
}
```

##### Record Type:
```bash
$ skycli record query country -o country.json --basedir=image
$ cat country.json
{"_id": "country/805b13a2-22c2-4785-87bf-5a179413f0e8", "image":"@file:image/someassetid-japan.jpg"}
{"_id": "country/720f5c7d-ac0b-46b3-88ae-969ae7ec37ea", "name": "United Kingdom"}
$ ls image
someassetid-japan.jpg
```

### Delete

#### Description
`skycli record delete` delete records from Skygear.

#### Synopsis

```bash
skycli record delete [options] <record_id> [<record_id> ...]
```

Record with ID `<record_id>` will be deleted from Skygear.

Use `skycli record delete --help` to view a list of available options.

#### Examples

```bash
$ skycli record delete city/f8cf1947-68ec-4fb0-9216-dc32ef92ddeb
```

### Set

#### Description
`skycli record set` set attributes of a record.

For each key-value pair in the command, the corresponding attributes of the record
will be replaced by the provided value. The other existing attributes of the record
will remain unchanged.

If the required record does not exist, it will be created with given attributes.

#### Synopsis

```bash
skycli record set [options] <record_id> <key>=<value> [<key>=<value> ...]
```

For record with ID `<record_id>`, its attribute `<key>` will be replaced by `<value>`.

Use `skycli record set --help` to view a list of available options.

#### Examples

```bash
$ skycli record set city/f8cf1947-68ec-4fb0-9216-dc32ef92ddeb name="Hong Kong"
```

### Get

#### Description
`skycli record get` fetch records from Skygear.

The result will be printed to stdout. If `-o` is specified, then the result will be stored with the given filename.

For each record with field containing `@asset:<assetID>`, the corresponding
asset will be downloaded. The file will be stored at the working directory,
unless `--basedir` is specified.

#### Synopsis

```bash
skycli record get [options] <record_id> [<record_id> ...]
```

Record with ID `<record_id>` will be fetched from Skygear.

Use `skycli record get --help` to view a list of available options.

#### Examples

##### Stdin:
```bash
$ skycli record get city/f8cf1947-68ec-4fb0-9216-dc32ef92ddeb student/bed763f3-071f-4d87-91fb-dccb22099162
{ "_id": "city/f8cf1947-68ec-4fb0-9216-dc32ef92ddeb", "name": "Hong Kong" }
{ "_id": "student/bed763f3-071f-4d87-91fb-dccb22099162", "name": "Alice", "age": 10}
```

##### Files and Base Directories:
```bash
$ skycli record get -o out.json --basedir=file --pretty-print
$ ls file
someassetid-hongkong.jpg
$ cat out.json
{
    "_id": "city/f8cf1947-68ec-4fb0-9216-dc32ef92ddeb",
    "name": "Hong Kong",
    "image": "@file:tmp/someassetid-hongkong.jpg"
}
```

### Get attribute

#### Description
`skycli record getattr` gets the value of an attribute of a record.

The value of the attribute will be printed to stdout.

If the attribute is `@asset:<assetID>`, the corresponding
asset will be downloaded. The file will be stored at the working directory,
unless `--basedir` is specified.

#### Synopsis

```bash
skycli record getattr [options] <record_id> <key>
```

The attribute `<key>` of the record with ID `<record_id>` will be fetched from Skygear.

Use `skycli record getattr --help` to view a list of available options.

#### Examples

##### Normal value:
```bash
$ skycli record getattr student/bed763f3-071f-4d87-91fb-dccb22099162 name
Alice
```

##### Asset:
```bash
$ skycli record getattr city/f8cf1947-68ec-4fb0-9216-dc32ef92ddeb image --basedir=file
@file:someassetid-hongkong.jpg
$ ls file
someassetid-hongkong.jpg
```

### Edit
#### Description
`skycli record edit` fetches a record from Skygear, ask user to edit it, and re-upload the record.

The record will be fetched, then an editor program will be opened including the record. After the editor is closed, the modified record will be uploaded.

If only the record type is specified, a new record with new ID will be created.

Note that removing an attribute in the editor WILL NOT remove the corresponding attribute in Skygear. The attribute will remain unchanged with the original value.

For uploading and downloading assets, please see `skycli record import` and `skycli record export`

#### Synopsis

```bash
skycli record edit [options] (<record_id>|<record_type>)
```

Use `skycli record edit --help` to view a list of available options.

#### Examples

##### Record ID:
```bash
$ skycli record edit student/bed763f3-071f-4d87-91fb-dccb22099162
<<Vim>>
{
    "_id": "student/bed763f3-071f-4d87-91fb-dccb22099162",
    "name": "Alice"
}
```

##### Record Type:
```bash
$ skycli record edit student
<<Vim>>
{
    "_id": "city/0123456",
}
```
