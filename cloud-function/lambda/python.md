---
title: Cloud code triggered by Client SDK
---

[[toc]]

You will need lambda functions if you want to have codes written on the backend
server instead of inside the SDK. The client SDK calls the cloud codes which
in turn return the response back to the client.

Typical usages include:

- processing payment transactions
- sending emails or SMS
- running complex database queries that cannot be done with
  the SDKs efficiently
- operations that requires permission checking from server side

A lambda function can be created using the `@skygear.op` decorator.

### Decorator Parameters

The decorator syntax is:

```python
@skygear.op(name, user_required=False, key_required=False)
```

- **`name`** (String)

  It is an identifier for the lambda function. The client SDKs call
  this function using this `name` identifier.

- **`user_required`** (boolean, optional)

  If `user_required` is set to `True`, only authenticated user
  can call this function. Skygear will return an `PermissionDenied`
  error if an unauthenticated user tries to call this function.

  The default value is `False`.

  You can obtain the user ID of the authenticated user by
  `skygear.utils.context.current_user_id()`.

- **`key_required`** (boolean, optional)

  If `key_required` is set to `True`, only authenticated user
  or client with API key can call this function. Skygear will return a
  `NotAuthenticated` error if an unauthenticated user and a client
  without API key tries to call this function.

  The default value is `False`. If `user_required` is set to `True`, this
  parameter is ignored.

### Passing arguments to Lambda Functions

The lambda function can accept arguments.
The list of arguments are defined in the function signature;
and the SDK must provide those arguments when calling the lambda
function. You can refer to the [example][lambda-example] for
how to pass arguments to lambda functions.

If a function takes no parameters, its signature should look like:

```python
@skygear.op('foo')
def my_func():
    pass
```

If a function takes two arguments, `song_name` and `singer`, its
signature should look like:

```python
@skygear.op('bar')
def my_func_two(song_name, singer):
    pass
```

Each of the arguments can be one of the following types.
They are similar to the available types in a JSON object.

- Number
- String
- Boolean
- Array
- Object

Depending on the SDKs, the supplied arguments will be
passed to the lambda function as the corresponding data type.

::: tips

**Tips:** In the JS SDK, the parameters are parsed using the `toJSON`
method. For example, a `Date` object in JS SDK will be passed to
the lambda function in Python as a string like `'2016-10-28T03:23:11.600Z'`.

:::

### Return Value

A lambda function should return a JSON-serializable Python dictionary or `None`.
It will be the response the client SDK receives.


### Example

The following lambda function, named `send_invitation_email`,
accepts two parameters: `to_user_email` and `custom_message`.
It sends the email using [SendGrid][sendgrid].

```python
import skygear
# need to include sendgrid in requirements.txt
import sendgrid


@skygear.op('send_invitation_email')
def send_invitation_email(to_user_email, custom_message):
    email = sendgrid.Mail()
    email.add_to(to_user_eail)
    email.set_from('admin@skygeario.com')
    email.set_subject('You are invited to try the app!')
    email.set_text(custom_message)

    client = sendgrid.SendGridClient('my_api_key')
    client.send(email)
    return {
      'result': 'OK',
    }
```

This function can be invoked from the client SDKs by the followings:

**JS SDK**

```javascript
const params = {
  'to_user_email': 'foo@bar.com',
  'custom_message': 'Hi there!',
};
skygear.lambda('send_invitation_email', params)
  .then(response => {
    console.log(response); // {'result': 'OK'}
  });
```

**Android SDK**

```java
Container skygear = Container.defaultContainer(this);

String lambdaName = "send_invitation_email";

// Argument order follows the lambda function signature
Object[] argv = new Object[]{ "foo@bar.com", "Hi there!" };

// Note: you can skip the argument when calling the function if there is none
skygear.callLambdaFunction(lambdaName, argv, new LambdaResponseHandler() {
    @Override
    public void onLambdaSuccess(JSONObject result) {
        // Your logic to handle the function result
    }

    @Override
    public void onLambdaFail(String reason) {
        // Error Handling
    }
});
```

**iOS SDK**

```obj-c
NSArray *argv = @[@"foo@bar.com", @"Hi there!"];
[container callLambda:@"send_invitation_email" arguments:argv completionHandler:^(NSDictionary *response, NSError *error) {
    if (error) {
        NSLog(@"error calling send_invitation_email:someone: %@", error);
        return;
    }

    NSLog(@"Received response = %@", response);
}];
```

```swift
let argv = ["foo@bar.com", "Hi there!"]
SKYContainer.default().callLambda("send_invitation_email", arguments: argv) { (response, error) in
    if let error = error {
        print("error calling send_invitation_email:someone: \(error)")
        return
    }
    print("Received response = \(response)")
}
```

[lambda-example]: #lambda-example
[sendgrid]: https://sendgrid.com
