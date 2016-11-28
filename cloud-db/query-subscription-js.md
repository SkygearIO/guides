---
title: Query Subscriptions
---

<a name="subscription"></a>
## Subscribe to Query Change (**Coming Soon**)

### Creating a subscription

The following code creates a subscription of all the `Note` created by
`skygear.currentUser`. Notice that creating a subscription does not involve
the use of the `new` keyword.

```javascript
const Note = skygear.Record.extend('note');
const query = new skygear.Query(Note);
query.equalTo('_created_by', skygear.currentUser.id);

subscription = skygear.Subscription('my notes');
subscription.query = query;
skygear.publicDB.saveSubscription(subscription);
```

### Fetching subscription

```javascript
skygear.publicDB.fetchSubscription('my notes').then((subscription) => {
  // examine the subscription here
}, (error) => {
  console.error('error fetching "my notes": %o', error);
});
```

### Fetching all subscriptions

```javascript
skygear.publicDB.fetchAllSubscriptions().then((subscriptions) => {
  subscriptions.forEach((subscription) => {
    // do something with the subscription here
  });
}, (error) => {
  console.error('error fetching all subscriptions: %o', error);
});
```

### Deleting a subscription

```javascript
skygear.publicDB.deleteSubscription('my notes').then((subscription) => {
  console.log('success deleting "my notes"')
}, (error) => {
  console.error('error deleting "my notes": %o', error)
});
```

### Listening to subscription notification

Once user subscribes, we can listen to subscription notifications.

```javascript
const listener = skygear.addNotificationListener((notification) => {
  if (notification.subscriptionID === 'my notes') {
    let id = notification.recordID;
    switch (notification.reason) {
      case skygear.NOTIFICATION_REASON_DELETED:
        // some note is deleted
        break;
      case skygear.NOTIFICATION_REASON_CREATED:
        // a new note is created
        break;
      case skygear.NOTIFICATION_REASON_UPDATED:
        // some note is updated with changes
        const changes = notification.recordChanges;
        // do something with the changes
        break;
    }
  }
});

// later in the program when done listening
listener.off();
```
