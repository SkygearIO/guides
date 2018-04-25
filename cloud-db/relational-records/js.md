---
title: Working with Relational Records
---

[[toc]]

## Creating a reference between records

Skygear supports parent-child relationship between records via _reference_.
`skygear.Reference` is a pointer class, which translates to foreign key in
skygear server database for efficient query.

You can even reference a user from a record. To learn more about user object or
how to retrieve user objects, you can read the
[User Authentication Basics][doc-auth-basics].
Notice that we are not using the `new` keyword when creating reference. Suppose you
have the user record object `rick`:

``` javascript
const note = new Note({
  heading: 'Working Draft',
  content: 'People involved please fill in',
});
const author = new skygear.Reference(rick);
note.author = author;
skygear.publicDB.save(note);
```

You can create reference between records:

``` javascript
const note1 = new Note({
  heading: 'Specification',
  content: 'This is first section',
});
const note2 = new Note({
  heading: 'Specification page 2',
  content: 'This is second section',
});
note1.nextPage = new skygear.Reference(note2);
skygear.publicDB.save([note2, note1]);
```

- The order of objects in batch save array matters if there are reference
relationship between records. In the above case, if you reverse the order
`[note1, note2]` will produce an error. `note1` will not be saved, while `note2` will be saved.

- To simultaneously retrieve `note1` and `note2` in one query,
you may use `transientInclude`. Read about it below.

## Querying referenced records

This example shows how to query all notes (`Note` record) who has an `account` field reference to a user record. In this example, we will query all notes where `account` equals to the current user.


``` javascript
const Note = skygear.Record.extend('note');
let query = new skygear.Query(Note);
query.equalTo('account', new skygear.Reference(skygear.auth.currentUser));

skygear.publicDB.query(query).then((r) => {
    console.log(r);
});

```

If you haven't have the corresponding record in hand (in this example, we will use the User record `182654c9-d205-43aa-8e74-d465c830087a`), you can reference with a specify `id` without making another query in this way:

``` javascript
const Note = skygear.Record.extend('note');
let query = new skygear.Query(Note);
query.equalTo('account', new skygear.Reference({
    id: 'user/182654c9-d205-43aa-8e74-d465c830087a'
}));

skygear.publicDB.query(query).then((r) => {
    console.log(r);
});

```

## Eager Loading

If you wish to retrieve `note1` and `note2` at the same time in one query,
you can perform eager loading using the transient syntax.
Here we have an example (notice that Delivery has reference to Address
on key `destination`):

``` javascript
const Delivery = skygear.Record.extend('delivery');
const Address = skygear.Record.extend('address');

const address = new Address({ /* some key-value pairs */ });
const delivery = new Delivery({ destination: new skygear.Reference(address) });
skygear.publicDB.save([address, delivery]);
```

Now if you want to query delivery together with the address:

``` javascript
const query = new skygear.Query(Delivery);
query.transientInclude('destination');
skygear.publicDB.query(query).then((records) => {
  console.log(records[0].destination);            // skygear.Reference object
  console.log(records[0].$transient.destination); // Address record object
}, (error) => {
  console.error(error);
});
```

You can also set an alias for transient-included field.

``` javascript
const query = new skygear.Query(Delivery);
query.transientInclude('destination', 'deliveryAddress');
skygear.publicDB.query(query).then((records) => {
  console.log(records[0].$transient.deliveryAddress); // Address record object
}, (error) => {
  console.log(error);
});
```

- It is possible to eager load records from multiple keys, but doing so
will impair performance
- If you have a record in the public database referencing a record in the
private database (or the other way around), `transientInclude` would fail
and give you `null` at the transient key. If you really need to do so, you
have to make another query.

## Deleting referenced records

For now, you have to delete the referencing record first and then the referenced record.

[doc-auth-basics]: /guides/auth/basics/js/
