---
title: Field-based ACL
---

[[toc]]

// Draft version

(Intro)

## ACL Discovery Level

- Queryable: Fields that can be used to perform any query predicates.
- Discoverable: Fields that can only be used to perform equality query predicate (= or IN operator, but not wrapping in NOT nor OR).
- NotQueryable: Fields that not available to perform query.

## Advanced ACL User Target of Field-based ACL

- Owner: the owner of the record instance
- Specific User: the user specified by user ID
- Dynamic User Set: the users specified in a field of the records
- Defined Roles: the user-defined roles
- Any Users: any logged-in users
- Public: any users with correct API key

## Default Field-based ACL Settings

## Setting Field-based ACL

### Portal

### HTTP Request

### SQL

## Record Partial Update

## Work together with record-based ACL

## Example: User Profile of a Social Application
