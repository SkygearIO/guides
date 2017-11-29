---
title: Custom HTTP Endpoint
---

[[toc]]

You can configure the cloud code as an HTTP handler, which can respond to
requests coming from outside the SDK. A custom HTTP endpoint can be
created using the `skygearCloud.handler()` decorator.

A custom HTTP endpoint can be useful for the followings:

- receiving requests from outside the Skygear SDK
- allowing a third party webhook to call upon (e.g. payment service)

## Defining a HTTP Handler

The decorator syntax is:

```javascript
handler(path: string, func: function(request: *, options: *): object, authRequired: Boolean, userRequired: Boolean)
```

- **`name` (String)**

  It specifies the URL path (following your Skygear server endpoint)
  that the cloud code will respond to. Some examples are shown below:

  ```javascript
  skygearCloud.handler('foo')
  // accepting:
  // - https://<your-end-point>/foo
  // not accepting:
  // - https://<your-end-point>/foo/
  // - https://<your-end-point>/foo/bar
  
  skygearCloud.handler('foo/')
  // accepting:
  // - https://<your-end-point>/foo/
  // - https://<your-end-point>/foo/a
  // - https://<your-end-point>/foo/a/
  // - https://<your-end-point>/foo/a/b
  // it will redirect to https://<your-end-point>/foo/ upon a request to:
  // - https://<your-end-point>/foo
  
  skygearCloud.handler('abc/def')
  // accepting:
  // - https://<your-end-point>/abc/def
  // this will override another handler of 'abc/' if it exists
  // not accepting:
  // - https://<your-end-point>/abc
  // - https://<your-end-point>/abc/
  ```

  When a route is not recognized, it will give you the response
  `{"result": {"status":"OK"}}`, which is the same as requesting
  directly to your Skygear endpoint.

- **`method` (List of Strings, optional)**

  It specifies the HTTP request methods allowed on the URL. Possible options are
  `GET`, `POST`, `PUT` and `DELETE`.

  The Skygear server will return an HTTP 404 error if the request method is
  not allowed.

  The default value is `['GET', 'POST', 'PUT']`.

  ::: tips

  **Tips:**

  You can create different functions to handle different request methods
  for the same endpoint, for example:
  
  ```javascript
  skygearCloud.handler('my_endpoint', (req) => {
  }, {
    method: ['GET']
  });
  
  skygearCloud.handler('my_endpoint', (req) => {
  }, {
    method: ['POST']
  });
  ```
  :::

- **`user_required` (boolean, default `False`)**

  Setting `user_required` to `True` means that the only authenticated users
  are allowed to access the endpoint.

  Since the handler accepts requests from third parties,
  for authentication purpose it needs to send an HTTP request with the
  headers `X-Skygear-API-Key` and `X-Skygear-Access-Token` containing
  your app's API Key and the authenticated user's access token.

  An example request using cURL is:

  ```
  curl -H "X-Skygear-API-Key: your-api-key" \
  -H "X-Skygear-Access-Token: user-access-token" \
  https://your-server-endpoint/your-handler-endpoint
  ```

  You can obtain the user ID of the authenticated user by:

  ```javascript
  skygearCloud.handler('your-handler-endpoint', (req, options) => {
    const {
      context
    } = options;
    return {
    	'key': req.url.query['key'],
    	'value': req.url.query['value'],
        'user_id': context.user_id
    };
  }, {
    userRequired: true
  });
  ```

## Return Value

You can return either of the followings in the handler.
The Skygear server will create the corresponding HTTP response to the request.

- a `String`

  Skygear will return an HTTP 200 response, with `Content-Type: text/plain`,
  where the string will be the response body.

- a JavaScript Object

  Skygear will return an HTTP 200 response, with
  `Content-Type: application/json`,
  where the response body will be a JSON-serialized representation
  of the value returned from the handler.

## Examples

To obtain the parameters passed through HTTP GET in the URL:

```javascript
// curl "https://<your-endpoint>/get_query_string?key=month&value=10"
skygearCloud.handler('get_query_string', (req) => {
  return {
  	'key': req.url.query['key'],
  	'value': req.url.query['value']
  };
}, {
  userRequired: false
});
```

To parse data from a `Content-Type=application/x-www-form-urlencoded` header:

```javascript
// curl -H "Content-Type: application/x-www-form-urlencoded" -d "a=3&b=at" https://<your-endpoint>/form
skygearCloud.handler('form', (req) => {
	req.form(function (formError, fields) {
		console.log(fields.a);
		console.log(fields.b);
	});
}, {
	userRequired: false
});
```

To parse data from a JSON request body:

```javascript
// curl -H "Content-Type: application/json" -d '{"name": "John"}' https://<your-endpoint>/json
skygearCloud.handler('json', (req) => {
	let body = JSON.parse(req.body);
	return { 'name': body.name};
}, {
	userRequired: false
});
```

To obtain the full request URL:

```javascript
// curl https://<your-endpoint>/url/more/levels?level=3
skygearCloud.handler('url/', (req) => {
	return { 'path': req.url.path};
}, {
	userRequired: false
});
```

[werkzeug-request-response]: http://werkzeug.pocoo.org/docs/wrappers/
[werkzeug-doc]: http://werkzeug.pocoo.org/docs/
