# Deploying your first Node.js server to Skygear with pre-configured template

## Introduction

Skygear offers pre-configured containerization templates so that developers can skip writing a Dockerfile when deploying. One of these templates is Node.js, which we will be using to write a microservice in this section.

## Prerequisites

Before you start this section, please ensure you have `skycli` installed and configured properly. If you haven't, follow [this](../set-up/set-up-steps.md) to set it up.

## Create a Skygear app

Create a directory called ****`nodeserver` ****and go inside:

```text
$ mkdir nodeserver && cd $_
```

### Create app and scaffold it with skycli

Enter the `skycli` app creation command and give the new app a name. It's better to prefix you app name with your name or preferred alias like your GitHub ID, since Skygear cluster is shared among everyone who has an account and it's likely someone has already taken the app name `nodeserver`.

Upon app creation success, app information such as its API endpoint and key will be listed. They are essential for API calling which we will be performing at later stages. You can either jot them down now or find them at [Skygear's Developer Portal](https://portal.skygear.dev/).

```text
 $ skycli app create
 ? What is your app name? <your_name>-nodeserver
 App name: <your_name>-nodeserver.
 Creating app...
 Your API endpoint: https://<your_name>-nodeserver.skygearapp.com/.
 Your Client API Key: <API Key>.
 Your Master API Key: <Master Key>.
 Created app successfully!
```

You will be asked a few questions on scaffolding the newly created app in your current directory.

Answer the first three questions with a Y:

```text
? Do you want to scaffold your app now? Or you can do it later by `skycli app scaffold` command.
Scaffold now? (Y/n) Y
```

```text
? You're about to initialze a Skygear app in this directory: <your-local-directory>
Confirm? (Y/n) Y
```

```text
? All files in <your-local-directory> would be DELETED. Confirm? (Y/n) Y
```

Pick `<your_name>-nodeserver`:

```text
? Select an app to be associated with the directory: (Use arrow keys)
❯ <your_name>-nodeserver
```

Select template `Node.js Express`:

```text
? Select template: (Use arrow keys)
  Empty
❯ Node.js Express
  Node.js MongoDB Blog App
  Node.js React/Express Fortune App
```

A success message will then be displayed:

```text
Success! Initialized "Node.js Express" template for <your_name>-nodeserver in <your-local-directory>.
```

## Some explanations just before deploying

Your directory should now look like this:

```text
.
├── README.md
├── frontend
│   ├── index.js
│   └── package.json
└── skygear.yaml  
```

A `skygear.yaml` is always needed in order to deploy. It's a configuration file carrying information about the deployment. Let's have a look at the one created by our Node.js Express template:

{% code title="./skygear.yaml" %}
```yaml
app: <your_name>-nodeserver      # your Skygear app name

deployments:
  - name: frontend               # an arbitrary name of the service
    type: http-service           # type of the service
    context: frontend            # root directory of your source code
    template: nodejs:12          # pre-configured template
    path: /                      # the path for the service
    port: 8080                   # the listening port of the service
```
{% endcode %}

The Skygear app name to be linked with this deployment is specified at line 1. A microservice named`frontend` is declared at line 4, with its properties lying underneath. Line 6 tells Skygear where the source code lies, while line 8 defines the endpoint to reach the service.

Skygear runs on an actual Kubernetes cluster which is container-based, still you might already notice there exists no Dockerfile in the directory frontend. This is because we have used a pre-configured template to build our Node.js program, as displayed at line 7. With a valid value given to the configuration filed `template`, you don't need to provide a Dockerfile anymore.

## Deploy the Skygear app

Ensure you are at the root of `nodeserver` where `skygear.yaml` lies, run:

```
$ skycli app deploy
```

Wait for the deployment to complete.

## **Test it out**

**Simply curl it:**

```bash
$ curl https://<your_name>-nodeserver.skygearapp.com/
Hello, world!
```

## Useful tips

* To exclude deploying specific files, create a `.skyignore` file. It works the same as `.dockerignore`.
* To deploy service on runtimes other than NodeJS, you can supply a custom `Dockerfile` instead. Learn about it [here](deploying-your-first-dockerfile-to-skygear.md).

