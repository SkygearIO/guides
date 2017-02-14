---
title: Skygear Chat Quick Start
---

The following quick start guide assumes that you are working on a new Node.js project with Skygear template project using Skygear cloud services.

The template project will configure the Skygear and the Skygear SDK for you so that you can start working on your app directly.

If you want to use Skygear Chat JS SDK with other JS projects (e.g. Ionic, React Native, etc), please check out the [Skygear JS Quickstart Guide](https://docs.skygear.io/guides/get-started/js/).

## Prerequisite

### 1. Create an account on [the Skygear portal](https://portal.skygear.io/signup)

Upon signing up, you can find the API keys and the server endpoints in _info_ page. You need them to configure your Skygear app.

### 2. Install Node.js

We need Node.js to run the template project. If you have never installed Node.js, install it as follow:

``` bash
# for MacOS
brew install node

# for Debian/Ubuntu
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
```
Visit [Node.js setup guide](https://nodejs.org/en/download/) for instructions if you are using a different operating system.

## Setting up the template project

Let's install and run the [Yeoman Skygear scaffolding generator](https://github.com/SkygearIO/generator-skygear) to set up the template project. React, Babel ES6 and Webpack will be bundle-installed.

Make sure you answer Y when you are asked to install the Skygear Chat extension.

``` bash
# Install yeoman and skygear generator globally
npm install -g yo
npm install -g generator-skygear

# Create your project folder
mkdir new-skygear-project
cd new-skygear-project

# Generate your project and follow on screen instruction
yo skygear

# Answer the following questions to configure Skygear
# You can find the API key and the endpoint in your Skygear portal
? What is your skygear endpoint? https://<your-app-name>.skygeario.com/
? What is your skygear API key <your-api-key>
? Do you want to install Skygear Chat extension? (The core SDK already got Auth, CloudDB, Push and Pubsub) (Y/n)

# Answer the following questions to set up the React.js and webpack environment
? Please choose your application name <your-app-name>
? Which style language do you want to use? css/sass/scss/less/stylus
? Enable postcss? yes/no

# Launch the template app
npm start
```
Now, open the browser and navigate to http://localhost:8000/. If the template project is set up successfully, you should see the following screen.

![JS Skygear Template Preview Screenshot](/assets/js/js-app-preview.png)

## What's in the template project

Before developing on top of the yo template, let's first take a look at what it has set up for you.

### 1. Skygear npm packages

The npm package `skygear` and `skygear-chat` have been installed together with the template project.

If you want to use them in your custom Node.js project, you can install it via
the terminal:

``` bash
npm install --save skygear
npm install --save skygear-chat
```

Then, in your JS file, import them.

```javascript
import skygear from 'skygear' // Skygear Auth, CloudDB, Push and Pubsub
import skygearchat from 'skygear-chat' // Skygear Chat
```

### 2. Skygear Configuration

For your client app to communicate with the Skygear server, you need to configure the API key and the app end point in the client.

In the template project, we have put the configuration in ```src/index.js``` and your app information in ```config/base.js```.

Codes in ```src/index.js#L9-15```:

```javascript
// Configure your Skygear app
skygear.config({
  apiKey: config.skygear.apiKey,
  endPoint: config.skygear.endPoint
}).then(() => {
  // Render the main component into the dom
  ReactDOM.render(<App />, document.getElementById('app'));
});
```

Codes in ```config/base.js#L4-9```:

```javascript
// Your app info here. You can get them from the Skygear portal
export default {
  'skygear': {
    'endPoint': 'https://<your_app_name>.skygeario.com/',
    'apiKey': '<your_app_api_key>'
  }
}
```

### 3. User authentication

A chat app requires a user authentication system.

In ```components/Main.js``` in the template project, you can find all the authentication related functions.

Codes in ```components/Main.js#L18-46```:

```javascript
//user login
doLogin() {
  let username = this.refs.username.value;
  let password = this.refs.password.value;

  skygear.loginWithUsername(username, password).then(() => {
    this.setState({
      'user': 'Logged in as '+username
    });
  }, () => {
    this.setState({
      'user': 'Login fails'
    });
  });
 }
//user signup
doSignup() {
  let username = this.refs.username.value;
  let password = this.refs.password.value;

  skygear.signupWithUsername(username, password).then(() => {
    this.setState({
      'user': 'Signup as ' + username
    });
  }, () => {
    this.setState({
      'user': 'Signup fails'
    });
  });
}
```

:::tips
 Even if your app does not require a user system, as every [record](https://docs.skygear.io/guides/cloud-db/basics/js/) in the Skygear database must be owned by a user, you need to at least create a user to perform CRUD. A user can be anonymous. More info at the [User Authentication Basics Guide](https://docs.skygear.io/guides/auth/basics/js/).
:::

## Enable Skygear Chat
Skygear Chat is disabled by default when you sign up a new Skygear account. To use the Skygear Chat APIs in your app, enable the Skygear Chat in the _plug-ins_ page in the [developer portal](https://portal.skygear.io/).

![Enable Chat Plugins at Portal
Screenshot](/assets/common/enable-chat-plugin-on-portal.png)

Hurray! You should be able to use Skygear Chat by now.

## Creating Your First Conversation

To see if everything is in a good place, let's add some codes to the template
project to create our first conversation (chat room).

In `src/index.js`, copy and paste the following codes so that we can sign up a few users
and create a conversation between them.

:::tips
Practically the codes should not be structured in that way. It is only for demo purpose.
:::

Your ```index.js``` file should look like this:

``` javascript
import 'core-js/fn/object/assign';
import React from 'react';
import ReactDOM from 'react-dom';

import skygear from 'skygear';
import skygearchat from 'skygear-chat';
import config from 'config';
import App from './components/Main';


skygear.config({
  apiKey: config.skygear.apiKey,
  endPoint: config.skygear.endPoint
}).then(() => {
  // Render the main component into the dom
  ReactDOM.render(<App />, document.getElementById('app'));

  skygear.signupWithUsername('UserA', '1234')
  .then(function(){
    return skygear.signupWithUsername('UserB', '1234');
  }).then(function(){
    return skygear.signupWithUsername('UserC', '1234');
  }).then(function(){
    return skygear.discoverUserByUsernames(
    ['UserA', 'UserB', 'UserC']);
  }).then((users) => {
    skygearchat.createConversation(
      users, // an Array of User Object indicating conversation participants
      'Group Chat of ABC', // a String for describing the conversation topic
    ).then((conversation) => {
      console.log('Conversation created!', conversation);
    }, (err) => {
      console.log('Fail to create the conversation', err);
    });
  });
});
```
If the codes run successfully, you should be able to see the conversation record in the Skygear Cloud DB. Open the Skygear data browser in the _info_ page in your [Skygear Portal](https://portal.skygear.io/).

![Open Database Web Browser
Screenshot](/assets/common/open-database-in-web-browser.png)

In the data browser, go to the table "conversation". If you can see a new record with the title "Group Chat of ABC", congratulation, the conversation is successfully created.

![Data Browser Result Screenshot](/assets/common/chat-data-browser-result.png)

Hurray! Everything should be in the right place from here. Check out the [Skygear chat documentation](https://docs.skygear.io/guides/chat/basics/js/) to start building your chat app.

## What's next from here

You got the basic! Next learn more about:
* [Skygear Chat](https://docs.skygear.io/guides/chat/basics/js/) or [API reference](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/index.js~SkygearChatContainer.html)
* [Skygear User Authentication](https://docs.skygear.io/guides/auth/basics/js/)
