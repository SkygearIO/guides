---
title: Error Handling
---

In addition to returning a result in your extension point, you can also return
an error to indicate to the client that an error has occurred. To do that,
you raise a `SkygearException` in your function.

```python
import skygear

@skygear.op("hello:world")
def have_error():
    raise SkygearException("error occurred")
```

When you raise an exception, Skygear Server and the SDK will be aware that
the operation is not successful. The error is passed along to the SDK
so that your app can handle the error.

## Overriding the default exception handler

It is possible to change the behavior of py-skygear so that you can
handle the exception before passing the error to Skygear Server. For example,
you can send an exception event to Sentry.

To do that, you use the `exception_handler` decorator:

```python
import raven
import skygear


client = raven.Client(dsn='https://public_key:secret_key@sentry.local/project_id')


@skygear.exception_handler(Exception)
def handle_error(exc):
    client.captureException()
```

If you would like to capture exception of another kind, you can specify
a different exception class to handle:

```python
@skygear.exception_handler(YourException)
def handle_error(exc):
    client.captureException()
```


You can capture exception of a certain kind and return another exception
instead:


```python
@skygear.exception_handler(YourException)
def handle_error(exc):
    return SkygearException("an exception has occurred")
```
