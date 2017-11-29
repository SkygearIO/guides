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

## Defining a Lambda

The method is:

```javascript
op(name: String, func: function(param: Object, options: *), authRequired: Boolean, userRequired: Boolean)
```

- **`name`** (String)

  It is an identifier for the lambda function. The client SDKs call
  this function using this `name` identifier.

- **`user_required`** (boolean, optional)

  If `user_required` is set to `true`, only authenticated user
  can call this function. Skygear will return an `PermissionDenied`
  error if an unauthenticated user tries to call this function.

  The default value is `false`.

  ```javascript
  skygearCloud.op('foo', function (param, options) {
    const {
      context
    } = options;
  	console.log(param['args']);
  	console.log(context.user_id);
  }, {
    userRequired: true
  });
  ```

- **`key_required`** (boolean, optional)

  If `key_required` is set to `true`, only authenticated user
  or client with API key can call this function. Skygear will return a
  `NotAuthenticated` error if an unauthenticated user and a client
  without API key tries to call this function.

  The default value is `false`. If `user_required` is set to `true`, this
  parameter is ignored.

## Passing arguments to Lambda Functions

The lambda function can accept arguments.
The list of arguments are defined in the function signature;
and the SDK must provide those arguments when calling the lambda
function. You can refer to the [example][lambda-example] for
how to pass arguments to lambda functions.

To retrieve the parameters passed from SDK, you can access the `args` properties from the JavaScript object.

```javascript
skygearCloud.op('foo', function (param) {
	console.log(param['args']);
});
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

## Return Value

A lambda function should return a JavaScript object or `null`.
It will be the response the client SDK receives.


## Example

The following lambda function, named `send_invitation_email`,
accepts two parameters: `to_user_email` and `custom_message`.
It sends the email using [SendGrid][sendgrid].

```javascript
// Remeber also install and save sendgrid as dependencies in package.json
var helper = require('sendgrid').mail;
var sg = require('sendgrid')('my_api_key');

skygearCloud.op('send_invitation_email', function(param) {
	var fromEmail = new helper.Email('admin@skygeario.com');
	var toEmail = new helper.Email(param['args']['to_user_email']);
	var subject = 'You are invited to try the app!';
	var content = new helper.Content('text/plain', param['args']['custom_message']);
	var mail = new helper.Mail(fromEmail, subject, toEmail, content);

	var request = sg.emptyRequest({
	  method: 'POST',
	  path: '/v3/mail/send',
	  body: mail.toJSON()
	});

	sg.API(request, function (error, response) {
		return {'result': 'OK'};
	});
})
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
