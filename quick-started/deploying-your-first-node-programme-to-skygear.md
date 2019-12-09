---
description: >-
  Skygear supports pre-configured Node.js containerization template so that
  developers do not have to write a Dockerfile when deploying a Nodejs
  Microservice to Skygear. Learn how it works.
---

# Deploying your first Node.js server to Skygear

Before you start this section, please ensure you have `skycli` installed and configured properly. 

## Create a Skygear app

**1. Create a directory** for the project.

```text
$ mkdir myProject && cd myProject
```

**2. Create a Skygear app.** Take notes of the API endpoint, the client API key, and the master API key of your newly created app as they are the identity of your app. Worry not if you forget the keys, you can always find them at the [Developer Portal](https://app.skygear.dev). 

```text
 $ skycli app create
 ? What is your app name? <app name>
 App name: <app name>.
 Creating app...
 Your API endpoint: https://<app name>.staging.skygearapp.com/.
 Your Client API Key: <API Key>.
 Your Master API Key: <Master Key>.
 Created app successfully!
```

**3. Set up the project using scaffold template.**`skycli` offers a couple of templates to help developers initialise their Skygear project quickly. In this example, we will use the `nodejs-example` template.

```text
 ? Do you want to setup the project folder now? Or you can do it later by `skycli app scaffold` command.
 Setup now? Yes
 ? You're about to initialze a Skygear Project in this directory: <directory>
 Confirm? Yes

 Fetching examples...
 ? Select example: nodejs-example

 Fetching js-example and initializing..
 Success! Initialized "nodejs-example" template in "<directory>".
```

Upon successfully initialising Skygear in the project directory, you should get a directory structure like this:

```bash
 --- <app root>
  |--- frontend          # the root of the 'frontend' service
  | |--- index.js
  | |--- package.json
  |--- skygear.yaml      # Skygear app configuration
```

## Configure the Skygear app

To successfully deploy a NodeJS program on Skygear, you need to write a simple configuration \(i.e. `skygear.yaml`\). Since we have prepared the configuration for you in the NodeJS scaffolding template, you do not have to write one before deploying the Hello World example. But you have to do so when you are writing your own Nodejs programme.

Here is a simple explanation of the configuration in the `skygear.yaml` .

```yaml
app: <app name>             # your app name, input during app creation
deployments:
  frontend:                 # an arbitrary name of the service
    type: http-service      # type of the service
    context: frontend       # the root directory of the service
    template: nodejs:12     # the deployment template to use
    path: /                 # the path for the service
    port: 8080              # the listening port of the service HTTP server
```

This configuration contains a microservice \(`frontend`\). The service will be deployed on NodeJS 12, at the root \(`/`\) of the app API endpoint.

## Deploy the Skygear app

**1. Run `skycli app deploy`** to deploy the Nodejs program.

```bash
$ skycli app deploy
```

You should see the message `Deployment completed` when the deployment is completes.

**2. Curl the API endpoint of the app** to ensure the deployment is successful.

```bash
$ curl https://<app name>.staging.skygearapp.com/
Hello, world!
```

Hurray! We are done.

## 

## Draft

1.  \(one to one mapping\)
   1.  \(one to one mapping\)
      1.  \(one to one mapping\)

## 

## 

## Useful tips

* To exclude deploying specific files, create a `.skyignore` file. It works the same as `.dockerignore`.
* To deploy service on runtimes other than NodeJS, you can supply a custom `Dockerfile` instead. Learn about it [here](https://app.gitbook.com/@ten-tang/s/skygear/~/drafts/-LuBuVmjRZJ6-T_Q_ur0/microserivces/deploying-your-first-dockerfile-to-skygear).

