---
title: Offline Storage
---

<a name="cached-query"></a>
## Cached Query

Skygear provides a simple cached query mechanism for application that wants to
display the data last seen before it gets new data from Skygear Server. To make
use of this, you just need to provide an extra callback function to the `query`
function, and Skygear will try to return an cached result before it gets the
latest result from the Skygear server.

``` javascript
const query = new skygear.Query(Note);
query.equalTo('title', 'First note');
function successCallback(notes, cached = false) {
  if (cached) {
    console.log('The result is a cached one');
  }
  console.log('Result', notes);
}
skygear.publicDB.query(query, successCallback)
.then(successCallback, (error) => {
  console.error(error);
});
```

- It is not guaranteed that the callback is called before a request takes
  place, especially if a cached result of the query is not available.
  In other words, if the result is not in the cache, the callback is only
  called once, after the requested result is returned from the server.
- It is guaranteed once the callback from server is called, cached callback will
  not get called.
- You may use one callback to handle both cached callback and server
  callback as above. But you are free to use two callbacks if it works
  better in your use cases.
