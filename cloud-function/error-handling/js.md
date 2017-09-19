---
title: Error Handling
---

[[toc]]

In addition to returning a result in your extension point, you can also return
an error to indicate to the client that an error has occurred. To do that,
you raise a `SkygearException` in your function.

```javascript
const SkygearError = skygearCloud.SkygearError;
skygearCloud.op('hello:world', function (food) {
	return new SkygearError("error occurred");
});
```

When you raise an exception, Skygear Server and the SDK will be aware that
the operation is not successful. The error is passed along to the SDK
so that your app can handle the error.

## Overriding the default exception handler

Not yet implemented in JS SDK.