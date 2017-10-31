---
title: Field-based ACL
---

[[toc]]

You can specify ACL to a record's field.
Similar to [Record-based ACL][doc-record-acl], you can set access level and
user target to Field-based ACL.

Field-based ACL is a more advanced feature of Skygear, it gives finer control
to the access, with [Discovery Level][doc-field-acl-discovery-level] and
[More User Target][doc-field-acl-user-target].

Field-based ACL also provide control on [Access Level][doc-access-level].

## Setting Field-based ACL

The simplest way to edit Field-based ACL is to use Skygear Developer Portal.

![Field ACL & Schema](/assets/common/field-acl-and-schema.png)

## ACL Discovery Level

- `Queryable`: Fields that can be used to perform any query predicates.

- `Discoverable`: Fields that can only be used to perform equality query predicate.
Supported operators: `equalTo`, `contains`(or `=` and `IN` for predicate in iOS).

- `Not Queryable`: Fields that not available to perform query.

## Advanced ACL User Target of Field-based ACL

- `Owner`: The owner of the record instance.

- `Specific User`: The user specified by user ID.

- `Fields with User Objects`: The users specified in a field of the records.

- `Role`: The user-defined roles.

- `Any Users`: Any logged-in users.

- `Public`: Any users with correct API key, including not authenticated users.

## Resource Matching

As mentioned in the [overview][doc-overview-resource], resource in Field-based ACL is defined by record type and record field.

Let's use the `${Record Type}:${Record Field}` as the format of Field-based ACL resource definition.

For example, you may define an ACL entry with resource `project:title`, that entry would affect the record field `title` of record type `project`. Record type `project` may have other fields, but the entry would only affect the `title` field.

You can also use wild-card `*` for record type and record field. Wild-card would match all of the record types or record fields.

There are 3 types of resource matching, with higher priority come first.

- `${Record Type}:${Record Field}`
- `${Record Type}:*`
- `*.*`

For example, when you try to do an operation on field `title` of record `project`. First the server would try to find entries for `project:title`. If no entry found, the server would then find `project:*`, and finally `*.*`. If not a single entry is matched, permission would be granted.

When there are entries that match the resource, server would NOT find fallback entries. The entries may not cover all user target, e.g. maybe only some roles or some users are specified. For user target that cannot be matched, they will get permission denied error for the operation.

:::tips
If you add entries that only match some users, the other users will be denied to perform operations for that field.

This is how the ACL setting form on Skygear Developer Portal works. When you choose `Private`, you would only see entry for `Owner`. You do not need to define entries for `Public` because all user except the owner would not match an entry and get denied for all operations.
:::

:::tips
If you choose `Default` on Skygear Developer Portal ACL setting form, you may see some entries in detail part, but there are actually no entries for the field. The display is showing the wild-card entries.
:::

## Record Partial Update

There are 2 cases when calling [record save API][doc-record-save]:

### atomic: `True`

If you change a field that you do not have access to write, those field will
not be changed, while others will be updated.

The SDK will provide partial update callback for record save API.

### atomic: `False`

You will simply get an error, for not having access right to update the field.

## Working Together with Access Level of Record-based ACL

The default is read and write access for public. So if you use Record-based ACL
**WITHOUT** Field-based ACL, you will be able to read all fields of a record as
long as you have read access to the record, and save all fields of a record as
long as you have write access to the record.

### Fetch and Query

They work on different level. Record-based ACL will affect **if you can, or cannot,
successfully get the record**, while Field-based ACL will affect **what you
can see** in the record.

Imagine that you set a record with public read write, and all the record's
field are no access to everyone. You will still get a list of records, but
you can only see the reserved fields of the records.

### Save

They work together similarly when you use save a record.

In atomic save, when a single record or a single field is failed to save,
the whole operation will be rolled back.

In non-atomic save, when a single record or a single field is failed to save,
that part would be skipped, and the rest of the operation will still succeed.


[doc-access-level]: /guides/cloud-db/acl-overview/js/#acl-access-level
[doc-record-acl]: /guides/cloud-db/record-acl/js/
[doc-field-acl-discovery-level]: /guides/cloud-db/field-acl/#acl-discovery-level
[doc-field-acl-user-target]: /guides/cloud-db/field-acl/#advanced-acl-user-target-of-field-based-acl
[doc-record-save]: /guides/cloud-db/basics/js/#saving-multiple-records
[doc-overview-resource]: /guides/cloud-db/acl-overview/js/#acl-resource
