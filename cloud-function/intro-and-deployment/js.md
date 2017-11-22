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

Under the hood, Cloud Functions communicate with the Skygear server
using a micro-services architecture through [ZeroMQ][zeromq]
or [HTTP2][http2].

Currently Skygear supports [Python 3][python3] and JavaScript for
Cloud Functions.

Below is a simple Cloud Functions example.
It is extracted from an app that stores information about our cats.
(Yes we got 4 cats in the office!)
The Cloud Functions below checks if the field "name" is filled before
saving the record to the database.
If not, it will raise an exception and will not save the record.


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

## Deploy Cloud Functions
<a name="deploy-cloud-functions"></a>
#### [Skycli][skycli-github] is the recommended way to deploy Skygear Cloud Functions.


`skycli` is the command line interface to Skygear Portal. You can use `skycli` to deploy your cloud functions to Skygear Cloud.

(Note: `skycli` for database schema editing is obsoleted. This guide is for the rewritten version to deploy cloud functions to Skygear Cloud.)

### Install Skycli

Firstly, you have to install `skycli` into your local machine. It only takes a few commands.

You can install Skycli via Homebrew (macOS) or NPM (macOS, Linux and Windows); You can also download a pre-built executable binaries.

#### Using Homebrew (for macOS)

```
$ brew tap skygeario/tap
$ brew install skycli
```

#### Using npm (for macOS, Linux and Windows)

```
$ npm install -g skycli
```

#### Download executable binaries

You can also download a pre-built executable binaries from [GitHub Releases][skycli-github-release]. We have pre-built executable binaries for macOS, Linux and Windows.


### Authenticate in Skycli

To init or deploy using `skycli`, you have to have an Skygear account (sign up for free at [Skygear Portal](https://portal.skygear.io/signup)).

#### Log in the command line tool

1. Open your command line interface. 
2. Run `skycli login`
3. You will be prompt your email and password. 

![Login to Skycli][skycli-login]
4. After logging in, you can then initialize or deploy your cloud code via skycli. The login session persist on your machine. If you want to switch to another account, you shall logout first.

#### Log out

1. Open your command line interface. 
2. Run `skycli logout`

![Logging out Skycli][skycli-logout]
3. You are now logged out on your local machine. To use `skycli deploy` and `skycli init`, you have to log into a Skygear account.


### Initialize a local project for Skygear Cloud

You can write cloud functions in Python or JavaScript to run custom functions on Skygear Server. Learn more about [Cloud Functions](https://docs.skygear.io/guides/cloud-function/intro-and-deployment/python/). 

Initialize an template project that helps you to get started with Skygear Cloud Function. Here is how you can do so.

1. Open your command line interface. Go to a destination for your project to init. e.g. `~/skyapp-cloud`
2. Run `skycli init`
3. **Select an app** - After confirmation on the project path, you will be asked which app to associate the local project with. (You can create a new app on [Skygear Portal](https://portal.skygear.io/))

![Init a cloud code folder][skycli-init]
4. **Whether to create static hosting directory** - Next, you can choose if you want to create your public static hosting directory. It is a directory for you to host assets or HTML files on Skygear statically.
5. **Select your language** - You will be prompted to choose use JavaScript or Python for your cloud code. A sample entry point will be set up (`index.js` for JavaScript and  `__init__.py` for Python).

![Logging out Skycli][skycli-choose-lang]
6. **Completed!** - skycli will then create files and configure your app info. **You can start to write your Cloud Functions now.**

![Init done][skycli-init-done]

### Deploying Cloud Function via skycli

Before using `skycli`, you have to push your code to master branch of Skygear Cloud Function git repository. Now you can just run `skycli deploy` in your project folder to get the code deployed.

It's recommended to use `skycli` to deploy Skygear Cloud Functions.

Here showing the steps to deploy to a sample app **skyapp**. We have written a hello function in index.js (JavaScript version):

![Sample cloud code][skycli-demo-cloud-code]

It's now ready to deploy this function to Skygear Cloud. 

1. Open your command line interface. Go to your project folder. e.g. `~/skyapp-cloud`
2. Check that you have updated your code and it's ready for deployment.
3. Run `skycli deploy`
4. **Skycli will archive your project and deploy.** You will see `Build completed successfully.` upon successful deployment.

![Deploying cloud functions][skycli-deploy]
5. Done! You should now see the new version of your Cloud Functions running.

### Updating skycli config

In the project folder, you can find `skygear.json` which contains config for the current project.

This is a sample for `skyapp` app:

```
{
  "version": 1,
  "app": "skyapp"
}
```

You can update `app` value to deploy skygear to a different app. Please make sure you are a collaborator of the specified project.

### Showing Skycli help menu

You can find out how to use `skycli` by running `skycli help`.

## Deploy Cloud Functions using Git

Note: `skycli` is the recommended way to deploy your cloud functions to Skygear Cloud. Please consider using `skycli` instead of Git, read [skycli guides][skycli-guide] for detail.

To have your Cloud Functions running on your Skygear server,
you need to deploy the Cloud Functions as a git repository to the Skygear cloud
server. After pushing your git
repository to the cloud, the codes will be deployed automatically.

These are the steps to deploy (push) Cloud Functions to the Skygear cloud:

1. get your code ready
2. configure your SSH public key in Skygear Portal
3. setup the git remote repository of Skygear Cloud
4. deploy the Cloud Functions

### 1. Get your code ready

You can write your own codes or use our [sample codes](https://gist.github.com/tszkanlo/f3ba96d185d0044b7542344298b9a0d9) to try out the deployment.

### 2. Configure your SSH key in Skygear Portal

You need to upload your SSH public key (`~/.ssh/id_rsa.pub`) to the
Skygear Portal through
[Manage Your Account][portal-manage-your-account].

Once you have set up the SSH key, you do not have to do it again for
other Skygear app projects in the same Skygear account.

If you have not generated an SSH key yet, you can follow
[this guide][ssh-key-guide] to generate one.

::: caution

**Caution:** If you have not uploaded your SSH public key or
you have uploaded an invalid one,
you will get the error message "Permission denied (publickey)"
when you try to push your git repository to Skygear Cloud.

:::



### 3. Set up the git remote repository of Skygear Cloud

In the command line, go to the sample code repository and run the following command to add a new git remote called `skygear-portal`:

```bash
git remote add skygear-portal ssh://<your-cloud-code-git-url>
# you can verify the remote repo list by `git remote -v`
```

You can obtain your Cloud Functions Git URL from the [Skygear Portal INFO tab][portal-app-info].


### 4. Deploy the Cloud Functions

Deploying your Cloud Functions to the Skygear Cloud can be done by
pushing them to the remote repository:

```bash
git push skygear-portal master
```

The Cloud Functions will then be deployed automatically.
It takes a few second to have your codes up and running on the server
after pushing them to the cloud.


A sample console output of a successful deployment is shown below.

```
MacBook-Pro:cloud-code bensonby$ git push skygear-portal master
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 495 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: Cloning into '/home/git/todo.git/build/tmp268340779'...
remote: done.
remote: Note: checking out '512709b0'.
remote:
remote: You are in 'detached HEAD' state. You can look around, make experimental
remote: changes and commit them, and you can discard any commits you make in this
remote: state without impacting any branches by performing another checkout.
remote:
remote: If you want to create a new branch to retain commits you create, you may
remote: do so (now or later) by using -b with the checkout command again. Example:
remote:
remote:   git checkout -b <new-branch-name>
remote:
remote: HEAD is now at 512709b... Add README
Starting build... but first, coffee!
2016-10-03 09:16:12,279 [__main__] Building your application image...

2016-10-03 09:16:12,284 [botocore.credentials] Found credentials in environment variables.
2016-10-03 09:16:12,416 [botocore.vendored.requests.packages.urllib3.connectionpool] Starting new HTTPS connection (1): s3-us-west-2.amazonaws.com
2016-10-03 09:16:26,041 [__main__] Pushing your image to the registry...


[Docker] The push refers to a repository [10.3.0.19:80/todo]
remote:  /
[Docker] git-512709b0: digest: sha256:6dc4e70bf2b1c0c3dd4daa535ab92d46da7a1fd7f022f28594fbbdd8b214f65d size: 3236
[Docker] Uploaded
remote: Size: 3236
remote: Digest:sha256:6dc4e70bf2b1c0c3dd4daa535ab92d46da7a1fd7f022f28594fbbdd8b214f65d
remote: Tag: git-512709b0
2016-10-03 09:16:53,527 [__main__] Done. Your application image is ready for deploy.

Build complete.
Launching app.
Launching...
Done, todo:v17 deployed to Deis

Use 'deis open' to view this application in your browser

To learn more, use 'deis help' or visit http://deis.io

To ssh://git@git.skygeario.com/todo.git
 + 66c7c40...512709b master -> master
```


Now it is time to verify the deployment.

You can test one of the 4 functions written in the sample codes,
the HTTP handler, by issuing a cURL command:

```bash
$ curl https://<your-skygear-endpoint>.skygeario.com/cat/feed
Meow! Thanks!
```
Do you see `Meow! Thanks!`? If yes it means the deployment is successful.
You are now good to go. :grinning:


## How Cloud Functions Works

Now, let's examine the Cloud Functions example in details.

Functions defined in the Cloud Functions `index.js` are recognized
by Skygear using decorators. There are 4 types of functions that
can be used; they are illustrated by the 4 functions in the examples.

### 1. Database Hooks

Database hooks can be used to intercept database-related API
requests. You can use the `before_save` hook to perform data validation,
or the `after_delete` hook to perform database clean-up tasks.

```javascript
// Reject empty 'name' before saveing a cat to the database
skygearCloud.beforeSave('cat', function(record, original, pool, options) {
    console.log(record["name"]);
    if (record["name"] === undefined || record["name"] === null || record["name"] === '') {
    	return new Promise((resolve, reject) => {
    		reject(new Error('Missing cat name'));
    	});
    }
    return;
}, {
    async: false
});
```

This example function will be called whenever a `cat` is saved
to the database, whether created or updated. When the new cat `record`
has an empty `name` attribute, it will raise an exception; and the
cat will not be saved.

More details can be found in
[Database Hooks][doc-cloud-code-db-hooks].

### 2. Scheduled Tasks

Functions decorated with `@skygear.every` are scheduled tasks that
work similar to the cron daemon in UNIX: they will be run
by the Skygear Cloud at a certain time or in regular intervals.

```javascript
// Cron job that runs every 2 minutes
skygearCloud.every('@every 2m', function() {
	// show 'Meow Meow!' every 2 minutes in Skygear Portal Production Log
	console.log('Meow Meow!');
});
```

This example function will run at two-minute intervals. You will be able to
see the log messages `'Meow Meow!'` in the
[Console Log][portal-console-log] of your app
in the Skygear Portal.

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

In this example, the HTTP handler is exposed through the URL
`https://<your-end-point>.skygeario.com/cat/feed`
as defined by `cat:feed`, accepting only `GET` requests.
It returns a response of the string `'Meow! Thanks!'`.

More details can be found in
[HTTP Handlers][doc-cloud-code-http-handler].


## Creating Cloud Functions from Scratch

All you need to create a Skygear Cloud Function project is a file called `index.js`. This file serves as the entry point of your Cloud Function and it must exist in the root directory of your repository.

```bash
$ mkdir cloudcode
$ cd cloudcode
$ git init
Initialized empty Git repository in /Users/skygear/cloudcode/.git/
$ touch index.js
$ git add index.js
$ git commit -m "Initial commit"
[master (root-commit) 1e6b5f5] Initial commit
 1 file changed, 0 insertions(+), 0 deletions(-)
  create mode 100644 index.js
$ git remote add skygear git@<url>:<app-name>.git
$ git push skygear master
```

To start writing the Cloud Functions using skygear decorators, you need to require
the `skygear` module, in `index.js` for example:

```
const skygearCloud = require('skygear/cloud');
```

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