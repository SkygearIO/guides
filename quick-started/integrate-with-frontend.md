# Host a blog site on Skygear \(Auth Gear, your frontend + backend + database\)

## Introduction

In this _Quick Start_ section, we will deploy a web-based blog consisted of:

* MongoDB database
* user system powered by Auth gear
* frontend written in React.js
* backend written in Node.js

Users of such app will need to log in before they can write and read blogs.

Source code of these components have been written for you already, as we will focus on how to integrate them together as one blog web app.

## Prerequisites

Before you start this section, please ensure you have `skycli` installed and configured properly. If not, follow [this](../set-up/set-up-steps.md) to set it up.

## Create a Skygear app

Enter the `skycli` app creation command and give the new app a name. It's better to prefix you app name with your name or alias like your GitHub ID, since Skygear cluster is shared among everyone who has an account and it's likely someone has already taken the app name `nodeblog`.

Upon app creation success, app information such as its API endpoint and key will be listed. They are essential for API calling which we will be performing at later stages. You can either jot them down now or find them at [Skygear's Developer Portal](https://portal.skygear.dev/).

```text
 $ skycli app create
 ? What is your app name? <your_name>-nodeblog
 App name: <your_name>-nodeblog.
 Creating app...
 Your API endpoint: https://<your_name>-nodeblog.skygearapp.com/.
 Your Client API Key: <API Key>.
 Your Master API Key: <Master Key>.
 Created app successfully!
```

## Connect to a database on MongoDB Atlas 

Finish this [_How to_](../how-to/how-to-connect-to-a-managed-database-in-skygear.md) and you are good to go. Ensure you have your Connection String stored as a Secret under the `nodeblog` app.

## Clone the project

```text
git clone https://github.com/skygear-demo/rcloud-examples.git
cd cloud-examples
```

We will do our work in nodejs-blog:

```text
cd nodejs-blog
```

Below is a directory tree of nodejs-blog: 

```text
.
├── README.md
├── backend
│   ├── api.js
│   ├── db.js
│   ├── index.js
│   ├── package-lock.json
│   └── package.json
├── frontend
│   ├── index.js
│   ├── package-lock.json
│   ├── package.json
│   └── static
│       ├── index.html
│       ├── list.html
│       ├── main.css
│       ├── main.js
│       └── write.html
└── skygear.yaml
```

## Replace the placeholders

### Fill in the actual app name in skygear.yaml

Replace `__APP_NAME__` with your app's name:

{% code title="./skygear.yaml" %}
```yaml
app: __APP_NAME__                  # <- HERE!

deployments:
  backend:
    type: http-service
    context: backend
    template: nodejs:12
    path: /api
    port: 8080
    environment:
    - name: MONGO_DB_URL
      secret: MONGO_DB_URL
  frontend:
    type: http-service
    context: frontend
    template: nodejs:12
    path: /
    port: 8080

hooks:
  - path: /api/after_user_create
    event: after_user_create
```
{% endcode %}

Upon deployment, `skycli` will look for a skygear.yaml at the current directory. Once located, it will deploy the directory and the code inside to Skygear under the app name provided at line 1 in `skygear.yaml`.

### Put the actual app endpoint and key in ./frontend/static/main.js

Replace `__API_ENDPOINT__` and `__API_KEY__` with app's endpoint and api key you have jotted down:

{% code title="./frontend/static/main.js" %}
```javascript
skygear.configure({
  endpoint: '__API_ENDPOINT__',      // <- HERE
  apiKey: '__API_KEY__',             // <- HERE 
});

...
```
{% endcode %}

We are initialising Skygear JS SDK here by supplying our app's information. Such SDK is developed allowing users to interact with Skygear services such as Auth gear through seamless and apparent methods.

## Manage users with Auth gear

Auth gear is a service that comes with Skygear. It has all features a modern auth server has to offer, meanwhile these functions can be access via direct API calls or Skygear SDKs. Every single component on a user's Skygear cluster can retrieve  user information from and interact with Auth gear, when access right is given. Auth gear takes care of all your needs on auth functions and features. From a security perspective, it also is battle-tested and has gone through rounds of security audit.

### Sign up / Log in

Time to see some code. If you are not already there, navigate to `./frontend/static/main.js`:

{% code title="./frontend/static/main.js" %}
```javascript
...

// starts at line 18
function signup(email, password) {
  skygear.auth.signup({ email: email }, password).then(
    function (user) {
      goToWriteBlogs();
    },
    function (err) {
      stopLoading();
      setSignupErrorMsg(err);
    }
  );
}

function login(email, password) {
  skygear.auth.login(email, password).then(
    function (user) {
      goToWriteBlogs();
    },
    function (err) {
      stopLoading();
      setSignupErrorMsg(err);
    }
  );
}
```
{% endcode %}

Through Skygear SDK, you can implement the functions like sign-up or log-in in your frontend by just calling methods provided by Skygear SDK. Both of `skygear.auth.signup` and `skygear.auth.login` take a tuple of email and password, which will be submitted Auth gear to perform sign-up and log-in operations.

The first parameter of the sign-up method takes an object, as we accept a variety of log-in IDs where email is just one of them. To name a few, email, username and phone number are some of our available log-in IDs.

Once a user has signed up or logged in, which we consider them as authenticated, s/he will be redirected to the write blog page via the execution of `gotToWriteBlogs()`. Otherwise, error message will be shown.

This is to give you a taste on how Auth gear can be integrated into an app, without tangling up the data flow.

### Log out

Often, a log-out operation can be achieved by deleting the user's access token and any other user information locally. In the world of Skygear, we recommend you to also call `skygear.auth.logout()` as we will invalidate the current access token. This prevents us from being attacked with reused access tokens.

{% code title="./frontend/static/main.js" %}
```javascript
...

// starts at line 56
$('#logout-btn').click(function () {
  skygear.auth.logout().then(function () {
    goToIndex();
  });
});
```
{% endcode %}

## Define the structure of an app - skygear.yaml

`skygear.yaml` is the primary configuration file telling Skygear how your app is formed with \(micro\)services, how they are declared and what resources such as Secrets and environment variables they will use.

Navigate to `./skygear.yaml`:

{% code title="" %}
```yaml
app: __APP_NAME__                  # should be already replaced

deployments:
  backend:
    type: http-service
    context: backend
    template: nodejs:12
    path: /api
    port: 8080
    environment:
    - name: MONGO_DB_URL
      secret: MONGO_DB_URL
  frontend:
    type: http-service
    context: frontend
    template: nodejs:12
    path: /
    port: 8080

hooks:
  - path: /api/after_user_create
    event: after_user_create

```
{% endcode %}

At line 6 and 15, we tell Skygear where the source code for backend and frontend locates. The values are the actual local directory name `backend` and `frontend`, while the same words at line 4 and 13 are the services' respective names. Them services will both be built with Skygear's pre-configured containerization templates `nodejs:12`, as declared at line 7 and 16. 

The value of field `path` will be the actual API endpoint others can reach the service. So the endpoint for service `frontend` will be `https://<your_name>-nodeblog.skygearapp.com/` whereas that for `backend` will be `https://<your_name>-nodeblog.skygearapp.com/api/`.

The Secret `MONGO_DB_URL` carrying the MongoDB's connection string is passed to the service `backend` as an environment variable, as shown in line 10-12.

Skygear also provides several hook events. In line 20-22 is a hook configuration on the event `after_user_create`, which will be triggered once a user is created on Auth gear. Code hooked to it is in `/api/after_user_create` which we will cover later.

## Interact with user-defined backend APIs

Up to this point, all samples and explanations were about Skygear features. How about the backend APIs I have written? How can I access them once the backend app is deployed?

The blog's API code is in `./backend/api.js`, which we won't be covering. They interact with the MongoDB and create/return corresponding blog payloads.

To see how these APIs can be called. Navigate to `./frontend/static/main.js`:

{% code title="./frontend/static/main.js" %}
```javascript
...

// starts at line 90
function fetchBlogs() {
  skygear
    .fetch('api/fetch_blogs', {
      method: 'POST',
      mode: 'cors',
      headers: {
        'Content-Type': 'application/json'
      }
    })
    ...
}
```
{% endcode %}

As you may recall, the `backend` service is registered in the Skygear app under the path `/api`. Another feature of Skygear JS SDK is the app's endpoint will automatically be prepended for you, so that you can call microservices directly by their path. The current user's access token along with some other headers and payloads will be substituted into the request for you as well.

As a result, to call a API called `fetch_blogs` in our backend service, we can simply use the wrapped fetch method with `api/fetch_blogs` as the first parameter.

This is how your microservices can be accessed through Skygear JS SDK. Of course they can also be reached via other methods like `curl`, but in that case you will have to fill in the headers and some required information yourself.

## Hook triggered after sign up

Web hooks are a mechanism to notify client-side application sent from the server-side when a new event has occurred. Say we are interested in storing the user information with a database owned by us, while Auth gear takes care of all the auth operations, this is made possible with Skygear's web hooks.

Navigate to `./backend/api.js`:

{% code title="./backend/api.js" %}
```javascript
const db = require('./db');

module.exports.afterUserCreate = async function (req, res) {
    const client = await db.connect();
    try {
        const user = req.body.payload.user;
        await db.saveUser(client, user);

        res.send(user);
    } finally {
        await client.close();
    }
};
```
{% endcode %}

Above is a function which stores a user's information into a database owned by the user, which is the managed MongoDB connected via `MONGO_DB_URL`.

As you might recall in `./skygear.yaml`:

{% code title="./skygear.yaml" %}
```yaml
...
...
#line 20
hooks:
  - path: /api/after_user_create
    event: after_user_create
```
{% endcode %}

The function is bound to the Skygear event `after_user_create` in the app's configuration. The user-storing functions will then get triggered when such event happens on Auth gear.

To see if such hook is working properly where a user is stored on your MongoDB, click on the "Collection" button of your database in the dashboard. You will find your database's records there.

## Deploy!

Time to find out how our blog works. Go to the root of nodejs-blog and enter:

```text
skycli app deploy
```

Wait for the deployment to complete, sign up and write some blogs.

