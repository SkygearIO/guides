---
title: Introduction and Deployment of Cloud Functions
---

[[toc]]


## What is Cloud Functions?

Cloud Functions are custom functions that run on the Skygear server.
They are useful when:

- you need to build functionalities that are not basic database operations
  provided in the SDK
- you do not want to expose your codes in the front end client

On Skygear.io, you can also host static website for your apps with Cloud Functions.

Under the hood, Cloud Functions communicate with the Skygear server
using a micro-services architecture through [ZeroMQ][zeromq]
or [HTTP2][http2].

Currently Skygear supports [Python 3][python3] and JavaScript for
Cloud Functions.

Below is a simple Cloud Functions example, it will be triggered before a `cat` record is saved, and log an error if `name` was not filled.


```javascript
// Reject empty 'name' before saving a cat to the database
const skygearCloud = require('skygear/cloud');
skygearCloud.beforeSave('cat', function(record, original, pool, options) {
    if (record["name"] == nil) {
    	console.log("error")
    }
    return;
}, {
    async: false
});
```
```python
# Reject empty 'name' before saving a cat to the database
@skygear.before_save('cat', async=False)
def validate_cat_name(record, original_record, db):
    if not record.get('name'):
        raise Exception('Missing cat name')
    return record
```

## Deploy Cloud Functions
<a name="deploy-cloud-functions"></a>
[skycli][skycli-github] is the recommended way to deploy Skygear Cloud Functions.

`skycli` is the command line interface to Skygear Portal. You can use `skycli` to deploy your cloud functions to Skygear Cloud.

The recommended way to install `skycli` is via npm:

```
$ npm install -g skycli
```

You may also install skycli via Homebrew (macOS) as follows or [download a binaries][skycli-github-release].

```
$ brew tap skygeario/tap
$ brew install skycli
```

You can run help to see the commands available at skycli:

```
$ skycli help
```

### Authenticate in skycli

To init or deploy using `skycli`, you need a Skygear account (sign up for free at [Skygear Portal](https://portal.skygear.io/signup)).

1. Open your command line interface. 
1. Run `skycli login`
1. You will be prompt your email and password. 
1. After logging in, you can then initialize or deploy your cloud code via skycli. The login session persist on your machine. If you want to switch to another account, you shall logout first.

![Login to Skycli][skycli-login]

::: tips
You can logout via `skycli logout`
:::

### Initialize a directory

You need to initialize a directory for a Skygear.io app endpoint before the deployment:

1. Open your command line interface. Go to a directory for your project to init. e.g. `~/skyapp-cloud`
2. Run `skycli init`
3. **Select an app** - After confirmation on the project path, you will be asked which app to associate the local directory with. (You can create a new app on [Skygear Portal](https://portal.skygear.io/))

![Init a cloud code folder][skycli-init]
4. **Whether to create static hosting directory** - Next, you can choose if you want to create your public static hosting directory. It is a directory for you to host assets or HTML files on Skygear.
5. **Select your language** - You will be prompted to choose use JavaScript or Python for your cloud code. A sample entry point will be set up (`index.js` for JavaScript and  `__init__.py` for Python).

![Choose skycli language][skycli-choose-lang]
6. **Completed!** - skycli will then create files and configure your app info. **You can start to write your Cloud Functions now.**

![Init done][skycli-init-done]

::: advanced
After skycli init, you will find a `skygear.json` in the root of project directory.
:::

### Deploy and upload Cloud Function

You can deploy by running `skycli deploy` in the directory initialized with a Skygear endpoint. You will see `Build completed successfully.` upon successful deployment.

![Deploying cloud functions][skycli-deploy]

::: advanced

If you prefer deploy and upload using `git push`, you can still upload your public key to Skygear
Portal via [Manage Your Account][portal-manage-your-account], and git push to the master branch of
the ssh git endpoint at your developer portal Cloud Functions page.

:::

## How Cloud Functions Works

Now, let's examine the Cloud Functions example in details.

Functions defined in the Cloud Functions `index.js` or `cloud_code.py` are recognized by Skygear using decorators. There are 4 types of functions that
can be used; they are illustrated by the 4 functions in the examples.

### 1. Database Hooks

Database hooks can be used to intercept database-related API
requests. For example, you can use the `before_save` hook to perform data validation, or the `after_delete` hook to perform database clean-up tasks.

```javascript
// Reject empty 'name' before saveing a cat to the database
skygearCloud.beforeSave('cat', function(record, original, pool, options) {
    if (!record['name']) {
    	return new Promise((resolve, reject) => {
    		reject(new Error('Missing cat name'));
    	});
    }
    return;
}, {
    async: false
});
```
```python
# Reject empty 'name' before saving a cat to the database
@skygear.before_save('cat', async=False)
def validate_cat_name(record, original_record, db):
    if not record.get('name'):
        raise Exception('Missing cat name')
    return record
```

This example function will be called whenever a `cat` is saved
to the database, whether created or updated. When the new cat `record`
has an empty `name` attribute, it will raise an exception; and the
cat will not be saved.

More details can be found in
[Database Hooks][doc-cloud-code-db-hooks].

### 2. Scheduled Tasks

Functions decorated with `@skygear.every` or `skygearCloud.every` are scheduled tasks that work similar to the cron daemon in UNIX: they will be run
by the Skygear Cloud at a certain time or in regular intervals.

```javascript
// Cron job that runs every 2 minutes
skygearCloud.every('@every 2m', function() {
	// show 'Meow Meow!' every 2 minutes in Skygear Portal Production Log
	console.log('Meow Meow!');
});
```
```python
# cron job that runs every 2 minutes
@skygear.every('@every 2m')
def meow_for_food():
    # Skygear Portal Console Log will show 'Meow Meow!' every 2 minutes
    log.info('Meow Meow!')
```

This example function will run at two-minute intervals. You will be able to
see the log messages `'Meow Meow!'` in the [Console Log][portal-console-log] of your app in the Skygear Portal.

More details can be found in
[Scheduled Tasks][doc-cloud-code-scheduled-tasks].

### 3. Lambda Functions

Lambda Functions allow you to implement custom APIs, which can
receive user-defined function arguments, and can be called from the SDK.
They are useful when you need codes that are not exposed in the front end,
e.g. connecting to a payment gateway to create transactions.

```javascript
// custom logic to be invoked from SDK, e.g.
// skygear.lambda('food:buy', {'food': 'salmon'}) for JS
skygearCloud.op('food:buy', function (food) {
	// TODO: call API about online shopping
	// Return an object to the SDK
	return Object.assign({success: true}, food["args"])
});
```
```python
# custom logic to be invoked from SDK, e.g.
# skygear.lambda('food:buy', {'food': 'salmon'}) for JS
@skygear.op('food:buy', user_required=False)
def buy_food(food):
    # TODO: call API about online shopping
    # return an object to the SDK
    return {
        'success': True,
        'food': food,
    }
```

This example function creates a custom API `food:buy`, which
takes a `food` as the argument. The lambda function
can be called from the SDK, e.g. using `skygear.lambda` from JS.

```javascript
const arguments = {'food': 'salmon'};
skygear.lambda('food:buy', arguments)
  .then(response => {
    console.log(response);
    // {'success': true, 'food': 'salmon'}
  })
  .catch(err => {
    console.error(err);
  });
```

More details can be found in
[Lambda Functions][doc-cloud-code-lambda].

### 4. HTTP Handlers

HTTP handlers can handle HTTP requests that are sent to the Skygear server
through the specified URLs. They are useful when you need to listen to external
requests like the webhook from a payment gateway service.

```javascript
// Accepting HTTP GET request to /cat/feed
skygearCloud.handler('cat:feed', (req) => {
  return "Meow! Thanks!\n";
}, {
  method: ['GET'],
  userRequired: false
});
```
```python
# accepting HTTP GET request to /cat/feed
@skygear.handler('cat:feed', method=['GET'])
def feed(request):
    # TODO: handle the request such as logging the request to database
    return 'Meow! Thanks!\n'
```

In this example, the HTTP handler is exposed through the URL
`https://<your-end-point>.skygeario.com/cat/feed`
as defined by `cat:feed`, accepting only `GET` requests.
It returns a response of the string `'Meow! Thanks!'`.

More details can be found in
[HTTP Handlers][doc-cloud-code-http-handler].

## Request Process Flowchart

The following flowchart summarizes the process for the Cloud Functions.

[![Cloud Functions Request Process Flowchart][doc-request-flow-chart]][doc-request-flow-chart]

[zeromq]: http://zeromq.org
[http2]: https://http2.github.io/
[python3]: https://docs.python.org/3/
[portal-manage-your-account]: https://portal.skygear.io/user/settings
[ssh-key-guide]: https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/
[portal-app-info]: https://portal.skygear.io/app/info
[doc-cloud-code-db-hooks]: /guides/cloud-function/database-hooks/js/
[portal-console-log]: https://portal.skygear.io/app/logs
[doc-cloud-code-scheduled-tasks]: /guides/cloud-function/scheduled-tasks/js/
[doc-cloud-code-lambda]: /guides/cloud-function/lambda/js/
[doc-cloud-code-http-handler]: /guides/cloud-function/http-endpoint/js/
[doc-request-flow-chart]: /assets/cloud-function/request-flow.svg
[skycli-github]: https://github.com/SkygearIO/skycli
[skycli-guide]: #deploy-cloud-functions
[skycli-github-release]: https://github.com/SkygearIO/skycli/releases
[skycli-choose-lang]:/assets/cloud-function/skycli-choose-lang.png
[skycli-demo-cloud-code]:/assets/cloud-function/skycli-demo-cloud-code.png
[skycli-deploy]:/assets/cloud-function/skycli-deploy.png
[skycli-init-done]:/assets/cloud-function/skycli-init-done.png
[skycli-init]:/assets/cloud-function/skycli-init.png
[skycli-login]:/assets/cloud-function/skycli-login.png
[skycli-logout]:/assets/cloud-function/skycli-logout.png
