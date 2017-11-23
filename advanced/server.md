---
title: Setup Skygear Development Server Locally
---

[[toc]]

This guide shows you how to set up a development [Skygear server](https://github.com/SkygearIO/skygear-server) on your local machine for development or custom deployment purpose.

::: caution
This guide is for advanced users only. Normally using Skygear.io should be the simplest way for development and production deployment.
:::

## Concept

A typical deployment of Skygear is like this:

```xml
                                                   +--------------------+
                                                   |     skygear-node   |
                      +------------------+    +--- |(JS Cloud Functions)|
                      |                  |    |    +------------------+-+
                      |                  |    |                       |
+----------------+    |                  |    |    +----------------+ |
|   (Optional)   |    |  skygear-server  |    |    |   py-skygear   | |
|     Nginx      |----|(1 to n instances)| ------- |    (python     | |
|  Load Balancer |    |                  |    |    |Cloud Functions)| |
+----------------+    |                  |    |    +----------------+ |
                      |                  |    |                 |     |
                      |                  |    |    +---------+  |     |
                      +------------------+    |    |  chat   |  |     |
                               |              +----|(Plugins)|  |     |
                               v                   +---------+  |     |
                         +----------+                   |       |     |
                         |          | ------------------+       |     |
                         |          |                           |     |
                         |PostgreSQL| --------------------------+     |
                         |          |                                 |
                         |          | --------------------------------+
                         +----------+
```

You can run multiple instances of skygear-server which provide core
functionality of Skygear. And also multiple instances of skygear-node and
py-skygear which are for Cloud Functions of JavaScript and Python
respectively. There are a number of official plugins, such as chat, which are
technically identical to other Cloud Functions.

All configuration are in environment for ease of deployment.

One module missing in the diagram above is the WebSocket server for pubsub.
Which is optional depends on if you need the pubsub functionality.

Connections between different services are either [zmq](http://zeromq.org/) or
HTTP.

For authentication token, the default token store is `JWT`. If you use `Redis` or
`FS`, you will need a Redis or file system available for skygear-server.

## Operating System

Skygear can be set up on:

* macOS
* Unix based machines
* Windows

To get Skygear Sever running, you can just install follow Part I to install
Skygear. If you wish to write cloud code in JavaScript, you will need to
continue setting up as Part II describes.

## Install Skygear Server

1. Set up PostgreSQL database in your local machine
    1. Mac version(easiest way): Install PostgreSQL with [postgresapp.com](http://postgresapp.com/)
1. Download skygear-server Binary from [GitHub Releases](https://github.com/SkygearIO/skygear-server/releases)
    1. **Mac**: Download `skygear-server-darwin-amd64` version
    1. **Unix**: Download the binary according to your architecture
1. Skygear is configured via environment variables. These are the minimal config to run Skygear.
	``` bash
        export DATABASE_URL="postgresql://postgres:@localhost/postgres?sslmode=disable"
        export API_KEY="changeme"
        export MASTER_KEY="secret"
        export APP_NAME="_"
        export TOKEN_STORE="jwt"
        export TOKEN_STORE_SECRET="jwt_secret"
	```
1. You can found the complete [environment variable list here](https://github.com/SkygearIO/skygear-server/blob/master/.env.example). Note that you may not need all environment variables to get your server set up.
1. Run skygear with command line tool. Skygear is running in your local machine. You can now config your app to connect to your local server.
1. [For Cloud Functions only] To enable JS cloud code plugin, you will need to add the follow environment variables in your **server environment**, so skygear will look for the skygear-node when it starts. Assume that you are running two skygear-node at `localhost:9000` and `localhost:9001`.
	``` bash
        export PLUGINS="JS1"
        export JS1_TRANSPORT="http"
        export JS1_PATH="http://localhost:9000"

        export PLUGINS="JS2"
        export JS2_TRANSPORT="http"
        export JS2_PATH="http://localhost:9001"
	```

## Install skygear-node

There could be multiple instances of Cloud Functions running as shown above. If
you wish to write cloud functions  in JS, you will need to install the node
runtime in your local machine as well. Here shows you how:

1. To get the node runtime, we need the Skygear SDK. Install Skygear SDK with npm:
	``` bash
	npm install skygear
	```
2. Set up these env variables in the command line tool that runs your **JS runtime environment**.
    ``` bash
	export PUBSUB_URL="ws://localhost:3000/pubsub"
	export SKYGEAR_ENDPOINT="http://localhost:3000"
	export HTTP="true"
	``` 
3. Run skygear-node in your project folder. Skygear will look for `index.js` as entry point and run your code
	``` bash
	$(npm bin)/skygear-node
	```

::: tips

If you want to configure multiple skygear-node, you may encounter the error
`http: multiple registrations for /static/`. This is because each skygear-node
by default try to serve static assets and register the same HTTP endpoint.

In that case, set the environment variable of `SERVE_STATIC_ASSETS` to `false`
at the instance that don't serve static asset.

:::

## Configurations of Skygear Server

As mentioned above, all configuration are via environment variables. Here are
some common features.

### App Name and API Key

It is strongly recommended that you change the App Name and the API Key.
The App Name should be a string consisting of alphanumeric and underscore
that can identify your app among other apps that might reside on the same
system. The App Name also serves as a prefix to the database schema, which
isolates data from the other apps.

The API Key serves as a shared secret between the Skygear backend and the
Skygear client app. The Skygear client app must possess the key to make API
requests to Skygear backend.

### S3 Configurations

Skygear supports using Amazon S3 as the default storage backend.
Set the AWS access key, secret key, region and bucket by the following
environment variables.

```
$ export ASSET_STORE=s3
$ export ASSET_STORE_PUBLIC=NO
$ export ASSET_STORE_ACCESS_KEY=AKXXXXXXXXXXXXXXXXXX
$ export ASSET_STORE_SECRET_KEY=3XXx/X/XxxXXXXxXxXxxXXXXxx00x0XXxXxx0xxX
$ export ASSET_STORE_REGION=us-east-1
$ export ASSET_STORE_BUCKET=thebucketname
```

If your S3 is readable by the public, you can set `ASSET_STORE_PUBLIC` to `YES`
so that the access signature can be omitted.

### Admin User

After launching Skygear Server, an administrative role (`Admin`) and an
admin user is created. The default username is `admin` and the default
password is `secret`.

Please remember to change the password of the admin user before your
application is push to production. The following Bash Script helps you to
change the password.

```bash
# Please update the following settings
USER_NAME='admin'
PASSWORD='secret'
NEW_PASSWORD='new-password'
SKYGEAR_API_KEY='changeme'
SKYGEAR_ENDPOINT='https://your-endpoint.skygeario.com/'


ACCESS_TOKEN=$(curl -sX POST \
-d "{
  \"action\": \"auth:login\",
  \"api_key\": \"$SKYGEAR_API_KEY\",
  \"username\": \"$USER_NAME\",
  \"password\": \"$PASSWORD\"
}" \
$SKYGEAR_ENDPOINT |
python -c 'import json,sys; print json.load(sys.stdin)["result"]["access_token"]'
) && \
curl -sX POST \
-d "{
  \"action\": \"auth:password\",
  \"api_key\": \"$SKYGEAR_API_KEY\",
  \"access_token\": \"$ACCESS_TOKEN\",
  \"old_password\": \"$PASSWORD\",
  \"password\": \"$NEW_PASSWORD\"
}" \
$SKYGEAR_ENDPOINT > /dev/null && \
echo "Success"
```

### Logging

Logging lets you keep track of what is happening in Skygear. Information
from log is very important for diagnosing bugs and performance issues.

List of logging levels: `debug`, `info`, `warn`, `error`, `fatal`, `panic`.

To change the Skygear logging level, modify `LOG_LEVEL`:

```
$ export LOG_LEVEL=warn
```

Skygear supports using [Sentry](https://getsentry.com/) as the logging backend.
To enable logging with Sentry, simply add `SENTRY_DSN` and `SENTRY_LEVEL`:

```
$ export SENTRY_DSN=http://abcd:efgh@sentry.example.com/sentry/project1
$ export SENTRY_LEVEL=warn
```

### All Environment Variables

* `APP_NAME` - string, alphanumeric or underscore, name of the application
* `API_KEY` - string, shared secret between backend and client app
* `MASTER_KEY` - string, for potentially destructive operations and request
  options restricted to system administrators
* `CORS_HOST` - string, hostname for cross origin access control
* `SLAVE` - boolean, specify `true` to run skygear-server is in
  slave mode.  If you have multiple skygear-servers running behind a load
  balancer, you should dedicate one skygear-server as leader and enable slave 
  mode on all the rest. When slave mode is enabled, features such as pubsub and 
  cron jobs are disabled on slaves so the slaves will not be in conflict with the
  leader, meaning that only one instance of skygear-server will be running these
  features.

#### Token store related
* `TOKEN_STORE` - the backing store for user token, supporting `fs`,
  `redis` and `jwt`. `fs` is intended for local development.
  Please use `redis` or `jwt` for production deployment.
  * `fs` - uses the file system to store access tokens
  * `redis` - uses Redis database to store access tokens
  * `jwt` - encodes user info in the access token passed to the client
* `TOKEN_STORE_PATH` - where the token will store when using `fs`. Or the
  URL when using `redis`, e.g. `redis://localhost:6379`
* `TOKEN_STORE_PREFIX` - string, prefix to the access token for using Redis.
* `TOKEN_STORE_EXPIRY` - integer, number of seconds the created access token
  will expire. Default is to never expire.
* `TOKEN_STORE_SECRET` - string, for `jwt`, the secret used to validates
  the access token.

#### Asset store related
* `ASSET_STORE` - the backing store of assets, currently supporting `fs` and `s3`.
  `fs` is intended for local development
* `ASSET_STORE_PUBLIC` - specifying the asset is public or follows ACL: `YES` or
  `NO`
* `ASSET_STORE_PATH` - where the asset is saved when using `fs` backing store.
* `ASSET_STORE_URL_PREFIX` - the URL prefix Skygear will generate for `fs`
  store.
* `ASSET_STORE_SECRET` - the signing key for `fs` store
* `ASSET_STORE_ACCESS_KEY` - `s3` access key
* `ASSET_STORE_SECRET_KEY` - `s3` secret key
* `ASSET_STORE_REGION` - `s3` region, with the default as `us-east-1`
* `ASSET_STORE_BUCKET` - `s3` bucket name
* `ASSET_STORE_S3_URL_PREFIX` - the URL prefix Skygear will generate for `s3`
  asset, it only works with public `s3` asset. Useful for CloudFront or custom
  domain setup.

#### Apple Push notification related
* `APNS_ENABLE` - string, `YES` or `NO`
* `APNS_ENV` - string, `sandbox` or `production`
* `APNS_CERTIFICATE` - full string of the certificate in PEM format
* `APNS_PRIVATE_KEY` - full string of the private key in PEM format
* `APNS_CERTIFICATE_PATH` - string, absolute path to the cert, in PEM format.
* `APNS_PRIVATE_KEY_PATH` - string, absolute path to the private key, in PEM format.

Read [this guide](http://stackoverflow.com/questions/991758/how-to-get-pem-file-from-key-and-crt-files)
to learn how to convert a cert/key to PEM.

#### GCM(Google Cloud Messaging) related
* `GCM_ENABLE` - string , `YES` or `NO`
* `GCM_APIKEY` - string, the API key you obtained from Google

## Docker Deployment

You can find Dockerfile and [`docker-compose.yml`](https://github.com/SkygearIO/skygear-server/blob/master/docker-compose.yml)
in most repository. Skygear also support using [.env](https://github.com/SkygearIO/skygear-server/blob/master/.env.example) to setup environments config.

For more information about how the `.env` file works, please read
https://github.com/joho/godotenv
