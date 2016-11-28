---
title: Custom HTTP Endpoint
---

You can configure the cloud code as an HTTP handler, which can respond to
requests coming from outside the SDK. A custom HTTP endpoint can be
created using the `skygear.handler` decorator.

A custom HTTP endpoint can be useful for the followings:

- receiving requests from outside the Skygear SDK
- allowing a third party webhook to call upon (e.g. payment service)

### Decorator Parameters

The decorator syntax is:

```python
@skygear.handler(name, method=['GET', 'POST', 'PUT'], user_required=False)
```

- **`name` (String)**

  It specifies the URL path (following your Skygear server endpoint)
  that the cloud code will respond to. Some examples are shown below:

  ```python
  @skygear.handler('foo') 
  # accepting:
  # - https://<your-end-point>/foo
  # not accepting:
  # - https://<your-end-point>/foo/
  # - https://<your-end-point>/foo/bar

  @skygear.handler('foo/') 
  # accepting:
  # - https://<your-end-point>/foo/
  # - https://<your-end-point>/foo/a
  # - https://<your-end-point>/foo/a/
  # - https://<your-end-point>/foo/a/b
  # it will redirect to https://<your-end-point>/foo/ upon a request to:
  # - https://<your-end-point>/foo

  @skygear.handler('abc/def') 
  # accepting:
  # - https://<your-end-point>/abc/def
  # this will override another handler of 'abc/' if it exists
  # not accepting:
  # - https://<your-end-point>/abc
  # - https://<your-end-point>/abc/
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

  <div class="tips">

  **Tips:**

  You can create different functions to handle different request methods
  for the same endpoint, for example:

  ```python
  @skygear.handler('my_endpoint', method=['GET'])
  def handle_get(request):
      pass

  @skygear.handler('my_endpoint', method=['POST'])
  def handle_post(request):
      pass
  ```

  </div>

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

  You can obtain the user ID of the authenticated user by
  `skygear.utils.context.current_user_id()`.

### Function Arguments

The function should take one argument, `request`,
which is a [`Request` object][werkzeug-request-response]
from the [werkzeug library][werkzeug-doc].

### Return Value

You can return either of the followings in the handler.
The Skygear server will create the corresponding HTTP response to the request.

- a `String`

  Skygear will return an HTTP 200 response, with `Content-Type: text/plain`,
  where the string will be the response body.

- a `dict`, a `list`, an `int`, or `None`

  Skygear will return an HTTP 200 response, with
  `Content-Type: application/json`,
  where the response body will be a JSON-serialized representation
  of the value returned from the handler.

- a `skygear.Response` object

  You can return a `skygear.Response` object, which takes
  the same parameters as the
  [werkzeug Response Object][werkzeug-request-response].

### Examples

To obtain the parameters passed through HTTP GET in the URL:

```python
# curl "https://<your-endpoint>/get_query_string?key=month&value=10"
@skygear.handler('get_query_string')
def json_request_handler(request):
    params = request.args
    return {
        'key': params['key'], # month
        'value': int(params['value']), # 10
    }
```

To parse data from a `Content-Type=application/x-www-form-urlencoded` header:

```python
# curl -H "Content-Type: application/x-www-form-urlencoded" -d "a=3&b=at" https://<your-endpoint>/form
@skygear.handler('form')
def json_request_handler(request):
    params = request.form
    return {
        'a': int(params['a']), # 3
        'b': params['b'], # at
    }
```

To parse data from a JSON request body:

```python
# curl -H "Content-Type: application/json" -d '{"name": "John"}' https://<your-endpoint>/json
@skygear.handler('json')
def json_request_handler(request):
    b = request.stream.read()
    s = b.decode(request.charset, request.encoding_errors)
    request_body = json.loads(s)
    name = request_body.get('name') # John
    return {
        'name': name,
    }
```

To obtain the full request URL:

```python
# curl https://<your-endpoint>/url/more/levels?level=3
@skygear.handler('url/')
def parse_url(request):
    # obtain request URL after url/
    # so you can perform further actions based on the request URL
    full_path = request.full_path # /url/more/levels?level=3
    path = full_path[len('/url/'):] # more/levels?level=3
    return {
        'path': path,
    }
```

[werkzeug-request-response]: http://werkzeug.pocoo.org/docs/wrappers/
[werkzeug-doc]: http://werkzeug.pocoo.org/docs/
