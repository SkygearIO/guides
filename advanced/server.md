---
title: Setup Skygear Development Server Locally
---

This guide shows you how to set up a development [Skygear server](https://github.com/SkygearIO/skygear-server) on your local machine for development or custom deployment purpose.

::: caution
This guide is for advanced users only. Normally using Skygear.io should be the simplest way for development and production deployment.
:::

Skygear can be set up on: 

* macOS
* Unix based machines
* Windows

To get Skygear Sever running, you can just install follow Part I to install
Skygear. If you wish to write cloud code in JavaScript, you will need to
continue setting up as Part II describes.

## Part I: Install Skygear Server**

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
1. [For Cloud Code only] To enable JS cloud code plugin, you will need to add the follow environment variables in your **server environment**, so skygear will look for the skygear-node when it starts. Assume that you are running skygear-node at `localhost:9000`.
	``` bash
        export PLUGINS="JS"
        export JS_TRANSPORT="http"
        export JS_PATH="http://localhost:9000"
	```

## **Part II: Install Skygear Cloud Code (JS)**

If you wish to write cloud code in JS, you will need to install the node runtime in your local machine as well. Here shows you how: 

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
	skygear-node
	```
