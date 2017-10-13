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

The simplest way to edit Field-based ACL is to use Portal.

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

## Record Partial Update

There are 2 cases when calling record save API:

### atomic: `True`

If you change a field that you do not have access to write, those field will
not be changed, while others will be updated.

The SDK will provide partial update callback for record save API.

### atomic: `False`

You will simply get an error, for not having access right to update the field.

## Working Together with Access Level of Record-based ACL

The default is read and write access for public. So if you use Record-based ACL
**WITHOUT** Field-based ACL, you will be able to read all fields of a record as
long as you have read access to the record.

### Fetch and Query

They work on different level. Record-based ACL will affect **if you can
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
[doc-record-acl]: /guides/cloud-db/record-acl/
[doc-field-acl-discovery-level]: /guides/cloud-db/field-acl/#acl-discovery-level
[doc-field-acl-user-target]: /guides/cloud-db/field-acl/#advanced-acl-user-target-of-field-based-acl
