---
title: Quick Start
description: Adding Skygear to your web apps
---

[[toc]]

## Try Skygear

If you are interested in having a sneak peek, you can play around with Skygear using this [code snippet](https://jsfiddle.net/0bu7urc6/). You can obtain your own server endpoints and API keys by signing up for a free Skygear account [here](https://portal.skygear.io/signup).

If you would prefer to use your own text editor, you can use this simple React [scaffolding](https://github.com/SkygearIO/generator-skygear) and follow the guide to set up your Skygear app. 

## Get started with JS Demo Projects

Alternatively, you might want to get started with Skygear using the following demo projects:
* [HTML5: Web Dinner Poll](https://github.com/skygear-demo/web-dinnerpoll): This is a simple dinner polling app demonstrating the usage of Skygear as a cloud database.
* [Ionic: Todo List](https://github.com/skygear-demo/ionic-todo-demo): This is a simple ToDo List app written in AngularJS demonstrating the usage of Ionic Framework and Skygear JS SDK.
* [React: Chat App](https://github.com/skygear-demo/react-chat-demo): This is a simple chat app demonstrating the usage of Skygear Server with the chat plugin.
* [Slackbot: Lunchbot](https://github.com/skygear-demo/skygear-lunchbot-js): This bot allows your Slack team to create a list of places for lunch and the bot will suggest a place to eat based on a schedule.
* You can view a full list of JS demos / tutorials at [github.com/skygear-demo](https://github.com/search?q=topic%3Askygear-js+org%3Askygear-demo)


## Install Skygear in your app

This guide will walk you through the steps to add Skygear to your HTML5 project and Node.js project from scratch.

### Prerequisite

- A Skygear account. Sign up [here](https://portal.skygear.io/signup).


### Step 1: Install the Skygear JS SDK and configure Skygear

There are two ways you might use Skygear JS SDK:
1. Use the SDK in HTML with CDN
1. Import the [npm package](https://www.npmjs.com/package/skygear)

#### Method 1: HTML5 project (CDN)

Add the following lines into the header of your HTML file.

```html5
<!--Skygear CDN-->
<script src="https://code.skygear.io/js/polyfill/latest/polyfill.min.js"></script>
<script src="https://code.skygear.io/js/skygear/latest/skygear.min.js"></script>

<!--Skygear configuration-->
<!--The app end point and the api key can be found in the developer portal-->
<script>
  skygear.config({
    'endPoint': 'https://<your-app-name>.skygeario.com/', // trailing slash is required
    'apiKey': '<your-api-key>',
  }).then(() => {
    console.log('skygear container is now ready for making API calls.');
  }, (error) => {
    console.error(error);
  });
</script>
```

:::note
You can get your server endpoints and the API keys in the _info page_ in your [developer portal](https://portal.skygear.io/apps) after signing up for the [Skygear Cloud Services](https://portal.skygear.io/signup).
:::

#### Method 2: Node.js project

1. Add Skygear to your Node.js project using npm install.

```bash
npm install skygear --save
```

2. Then require it from any JavaScript file:

```javascript
var skygear = require('skygear');
```
If you are using ES2015, you can import the module instead:

```javascript
import skygear from 'skygear';
```

3. To configure Skygear in your app , add the following lines in the JS file.
```javascript
skygear.config({
  'endPoint': 'https://<your-app-name>.skygeario.com/', // trailing slash is required
  'apiKey': '<your-api-key>',
}).then(() => {
  console.log('skygear container is now ready for making API calls.');
}, (error) => {
  console.error(error);
});
```
:::note
You can get your server endpoints and the API keys in the _info page_ in your [developer portal](https://portal.skygear.io/apps) after signing up for the [Skygear Cloud Services](https://portal.skygear.io/signup).
:::

That is it. You are good to go now. You can continue with the step below to learn about basic database operations.

### Step 2: Create your first record in Skygear

Now, let's create a record in the Skygear database to see if the SDK has been installed successfully.

Add the following lines after the `skygear.config` function using promise to create a record. This is how the codes should be placed.

:::tips
Practically the codes should not be structured this way. It is for demo only.
:::

```javascript
skygear.config({
  'endPoint': 'https://<your-app-name>.skygeario.com/', // trailing slash is required
  'apiKey': '<your-api-key>',
}).then(function() {
  // Every record in Skygear must be owned by a user
  // For testing purpose, we have used signupAnonmously to create a record
  // Visit the user authetication documentation to learn more
  // https://docs.skygear.io/guides/auth/basics/js/
  return skygear.auth.signupAnonymously()
}).then(function() {
  // Create Record Type "test" and put "Hello world" as value of key "content"
  // Advanced: Skygear Server will create a table "test" and appropriate
  //           columns in PostgreSQL in Development mode.
  var Test = skygear.Record.extend('test');
  return skygear.publicDB.save(new Test({
    'content': 'Hello World'
  }));
}).then(function(record) {
  console.log('Record saved');
}).catch(function(err) {
  console.error('Error: ' + err.message);
});
```

If the record is created successfully, you should see the record "Hello World" in your database table "test".

You can access your database using the data browser we provide. It can be found from the _info_ page in your [developer portal](https://portal.skygear.io/apps).

![Skygear portal](/assets/common/open-database-in-web-browser.png)

This is how your data browser will look like.
![Web database viewer](/assets/common/quickstart-database-viewer.png)

:::tips 
<a id="tips-anchor"></a>
You can access Skygear database in 3 ways.
1. Web data browser: It can be found from the _info_ page in your [developer portal](https://portal.skygear.io/apps).
2. PostgreSQL client: Skygear database can be viewed in any PostgreSQL client. Get the connection string from the _info_ page in your [developer portal](https://portal.skygear.io/apps). We recommend using [Postico](https://eggerapps.at/postico/).
3. Skygear CMS: Skygear CMS is a business-user friendly web interface that allows users to edit the data in the database. To use the CMS, you have to enable it in the _plug-ins_ page in the [developer portal](https://portal.skygear.io/apps). Your CMS URL is `https://insert-your-app-name.skygeario.com/cms`.

:::

Hurray! Everything should be in the right place from here.

## What is next?
Next, you may want to learn more about:
* [Skygear Cloud Database basics](https://docs.skygear.io/guides/cloud-db/basics/js/)
* [Skygear User Auth Introduction](https://docs.skygear.io/guides/auth/basics/js/)
* [Skygear Chat Introduction](https://docs.skygear.io/guides/chat/basics/js/)
* [Other Skygear Guides](https://docs.skygear.io/)
