---
title: Cloud code triggered by Client SDK
---

[[toc]]

You will need lambda functions if you want to have codes written on the backend
server instead of client side. The client SDK calls the cloud codes which
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

## Passing arguments to Lambda Functions

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

In addition to the JSON-compatible data types above, the following Skygear
data type can be used in parameters as well. Serialization takes place
automatically and you get these objects on the cloud code side.

- Date
- Location
- Reference
- Asset
- Record

Note: In JavaScript, the SDK upload Asset (such as image) before calling your
cloud code. On other SDK, please upload the Asset before calling the lambda.

Objects will be deserialized according to the platform. For example, Array
will be serialized to `List` in Android and `NSArray` in iOS.

## Return Value

A lambda function can also return a value. The list of supported data types
are the same with parameters. That means you can return a date, a location
or an array.

## Examples

### Send an email

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

### Find locations

```python
import skygear
from skygear.models import Location


@skygear.op('find_locations')
def find_locations(current, keyword):
  // current - a Location
  // keyword - a String

  // Search in the database or call external service to find a list of
  // locations.
  return [
    {
      "loc": Location(22.2855200, 114.1576900),
      "name": "ACME Restaurant"
    },
    {
      "loc": Location(25.105497, 121.597366),
      "name": "ACME Restaurant"
    }
  ]
```

This function can be invoked from the client SDKs by the followings:

**JS SDK**

```javascript
const params = {
  'current': new skygear.Geolocation(13.444304, 144.793732),
  'keyword': 'ACME',
};
skygear.lambda('find_locations', params)
  .then(response => {
    console.log(response[0].name); // 'ACME Restaurant'
  });
```

**Android SDK**

```java
Container skygear = Container.defaultContainer(this);

String lambdaName = "find_locations";

Location current = new Location("skygear");
current.setLatitude(13.444304);
current.setLongitude(144.793732);

// Argument order follows the lambda function signature
Object[] argv = new Object[]{ current, "ACME" };

// Note: you can skip the argument when calling the function if there is none
skygear.callLambdaFunction(lambdaName, argv, new TypedLambdaResponseHandler<List<Map<String, Object>>>() {
    @Override
    public void onLambdaSuccess(List<Map<String, Object>> result) {
        LOG.i("MyApplication", result.get(0).get(name)); // 'ACME Restaurant'
    }

    @Override
    public void onLambdaFail(String reason) {
        // Error Handling
    }
});
```

```kotlin
val container = Container.defaultContainer(this)
val current = Location("skygear");
current.latitude = 13.444304
current.longitude = 144.793732

container.callLambdaFunction("find_locations", arrayOf<Any>(current, "ACME"), object : TypedLambdaResponseHandler<List<Map<String, Object>>>() {
    override fun onLambdaSuccess(result: List<Map<String, Object>>) {
        LOG.i("MyApplication", result.get(0).get("name")); // 'ACME Restaurant'
    }

    override fun onFail(error: Error?) {

    }
})
```

**iOS SDK**

```obj-c
NSArray *argv = @[
    [[CLLocation alloc] initWithLatitude:22.3 longitude:114.2],
    @"ACME"
];
[container callLambda:@"find_locations" arguments:argv completionHandler:^(id response, NSError *error) {
    if (error) {
        NSLog(@"error calling find_locations: %@", error);
        return;
    }

    NSLog(@"Name %@", [[response objectAtIndex:0] objectForKey:@"name"]);
}];
```

```swift
let argv = [CLLocation(latitude: 22.3, longitude: 114.2), "ACME"]
SKYContainer.default().callLambda("find_locations", arguments: argv) { (response, error) in
    if let shops = response as? [Dictionary<String,Any>] {
        print("Name: \(shops[0]["name"])")
    }
}
```

[lambda-example]: #lambda-example
[sendgrid]: https://sendgrid.com
