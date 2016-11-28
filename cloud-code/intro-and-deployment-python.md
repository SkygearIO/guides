---
title: Introduction and Deployment of Cloud Code
---

<a name="what-is-cloud-code"></a>
## What is Cloud Code?

Cloud codes are custom functions that run on the Skygear server.
They are useful when:

- you need to build functionalities that are not basic database operations
  provided in the SDK
- you do not want to expose your codes in the front end client

Under the hood, cloud codes communicate with the Skygear server
using a micro-services architecture through [ZeroMQ](http://zeromq.org)
or [HTTP2](https://http2.github.io/).

Currently Skygear supports [Python 3](https://docs.python.org/3/) for
cloud codes. JavaScript is on the roadmap.

Below is a simple cloud code example.
It is extracted from an app that stores information about our cats.
(Yes we got 4 cats in the office!)
The cloud code below checks if the field "name" is filled before
saving the record to the database.
If not, it will raise an exception and will not save the record.

```python
# Reject empty 'name' before saving a cat to the database
@skygear.before_save('cat', async=False)
def validate_cat_name(record, original_record, db):
    if not record.get('name'):
        raise Exception('Missing cat name')
    return record
```

<a name="deploying-to-skygear-cloud"></a>
## Cloud code deployment


To have your cloud codes running on your Skygear server,
you need to deploy the cloud codes as a git repository to the Skygear cloud
server. After pushing your git
repository to the cloud, the codes will be deployed automatically.

These are the steps to deploy (push) cloud codes to the Skygear cloud:

1. get your code ready
2. configure your SSH public key in Skygear Portal
3. setup the git remote repository of Skygear Cloud
4. deploy the cloud codes

### 1. Get your code ready

You can write your own codes or use our sample codes to try out the deployment.

Download the sample codes [here](https://github.com/skygear-demo/cloud-code-quick-start) or simply run this in your command line:

```
git clone https://github.com/skygear-demo/cloud-code-quick-start.git
```

The sample repository contains 3 files:

- `__init__.py` - to make this a package directory and import the cloud code
- `cloud_code.py` - where the cloud codes are
- `README.md` - the README file


### 2. Configure your SSH key in Skygear Portal

You need to upload your SSH public key (`~/.ssh/id_rsa.pub`) to the
Skygear Portal through
[Manage Your Account](https://portal.skygear.io/user/settings).

Once you have set up the SSH key, you do not have to do it again for
other Skygear app projects in the same Skygear account.

If you have not generated an SSH key yet, you can follow
[this guide](ssh-key-guide) to generate one.

<div class="caution">

**Caution:** If you have not uploaded your SSH public key or
you have uploaded an invalid one,
you will get the error message "Permission denied (publickey)"
when you try to push your git repository to Skygear Cloud.

</div>



### 3. Set up the git remote repository of Skygear Cloud

In the command line, go to the sample code repository and run the following command to add a new git remote called `skygear-portal`:

```bash
git remote add skygear-portal ssh://<your-cloud-code-git-url>
# you can verify the remote repo list by `git remote -v`
```

You can obtain your Cloud Code Git URL from the [Skygear Portal INFO tab](https://portal.skygear.io/app/info).


### 4. Deploy the cloud codes

Deploying your cloud codes to the Skygear Cloud can be done by
pushing them to the remote repository:

```bash
git push skygear-portal master
```

The cloud code will then be deployed automatically.
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

<a name="how-cloud-code-works"></a>
## How Cloud Code Works

Now, let's examine the cloud code example in details.

Functions defined in the cloud code `cloud_code.py` are recognized
by Skygear using decorators. There are 4 types of functions that
can be used; they are illustrated by the 4 functions in the examples.

### 1. Database Hooks

Database hooks can be used to intercept database-related API
requests. You can use the `before_save` hook to perform data validation,
or the `after_delete` hook to perform database clean-up tasks.

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
[Database Hooks](/cloud-code/guide/triggering-cloud-code#database-triggers).

### 2. Scheduled Tasks

Functions decorated with `@skygear.every` are scheduled tasks that
work similar to the cron daemon in UNIX: they will be run
by the Skygear Cloud at a certain time or in regular intervals.

```python
# cron job that runs every 2 minutes
@skygear.every('@every 2m')
def meow_for_food():
    # Skygear Portal Console Log will show 'Meow Meow!' every 2 minutes
    log.info('Meow Meow!')
```

This example function will run at two-minute intervals. You will be able to
see the log messages `'Meow Meow!'` in the
[Console Log](https://portal.skygear.io/app/logs) of your app
in the Skygear Portal.

More details can be found in
[Scheduled Tasks](/cloud-code/guide/triggering-cloud-code#scheduled-tasks).

### 3. Lambda Functions

Lambda Functions allow you to implement custom APIs, which can
receive user-defined function arguments, and can be called from the SDK.
They are useful when you need codes that are not exposed in the front end,
e.g. connecting to a payment gateway to create transactions.

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
[Lambda Functions](/cloud-code/guide/triggering-cloud-code#lambda-functions).

### 4. HTTP Handlers

HTTP handlers can handle HTTP requests that are sent to the Skygear server
through the specified URLs. They are useful when you need to listen to external
requests like the webhook from a payment gateway service.

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
[HTTP Handlers](/cloud-code/guide/triggering-cloud-code#http-endpoint).

<a name="creating-cloud-code-from-scratch"></a>
## Creating Cloud Code from Scratch

At a minimum, you need a git repository with the `__init__.py` file
for the cloud code to initialize itself.
You can import other files from `__init__.py` as seen in the example we used.

The following bash commands create a `cloud-code` git repository directory with a
`__init__.py` file importing codes from `hello_world.py`, which is ready for
a "hello world" cloud code deployment.

```bash
# create and initialize new git repository `cloud-code`
mkdir cloud-code
cd cloud-code
git init

# use hello_world.py to write the first cloud code
echo "from .hello_world import *" > __init__.py
touch hello_world.py

# create requirements.txt for managing package dependencies
echo "# list your Python package dependencies" > requirements.txt

# commit the changes
git add .
git commit -m "Initial commit"
```

To start writing the cloud code using skygear decorators, you need to import
the `skygear` module, in `hello_world.py` for example:

```
import skygear
```

Package dependencies are managed by [pip](https://pip.pypa.io/en/stable/).
Upon deployment, any packages listed in `requirements.txt`
will be automatically installed.

Tips: You do not need to specify `skygear` in `requirements.txt`. It is
automatically installed for you.

<a name="request-process-flowchart"></a>
## Request Process Flowchart

The following flowchart summarizes the process for the cloud codes.

[![Cloud Code Request Process Flowchart](/assets/cloud-code/request-flow.svg)](/assets/cloud-code/request-flow.svg)

[ssh-key-guide]: https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/
