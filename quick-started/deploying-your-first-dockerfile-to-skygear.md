# Deploy a Dart server with self-defined Dockerfile

## Prerequisites

Before you start this section, please ensure you have `skycli` installed and configured properly. 

## Introduction

If the language you are coding in isn't on our pre-configured containerization template list or you wish to have more control on the containerization process, write your own Dockerfile and Skygear will follow  instructions inside to build your container image.

In this quick start guide, we will build a simple server written in Dart, which unfortunately isn't one of our pre-configured templates yet.

## Create a Skygear app

**1. Create a directory called `dartserver` and go inside**:

```text
$ mkdir dartserver && cd $_
```

**2. Create a Skygear app.** Enter the `skycli` app creation command and give the new app a name. It's better to prefix you app name with your name or alias like your GitHub ID, since Skygear cluster is shared among everyone who has an account and it's likely someone has already taken the app name `dartserver`.

Upon app creation success, app information such as its API endpoint and key will be listed. They are essential for API calling which we will be performing at later stages. You can either jot them down now or find them at Skygear's Developer Portal. \(\_\_TODO\_\_: confirm this link and those below\)

```text
 $ skycli app create
 ? What is your app name? <your_name>-dartserver
 App name: <your_name>-dartserver.
 Creating app...
 Your API endpoint: https://<your_name>-dartserver.skygearapp.com/.
 Your Client API Key: <API Key>.
 Your Master API Key: <Master Key>.
 Created app successfully!
```

**3. Write some Dart code.** Well, we have the Dart code ready. Paste the below command block into you terminal, which will create a `main.dart` with a simple "Hello World!" API implemented.

```text
$ echo 'import "dart:io";

Future main() async {
  // #docregion bind
  var server = await HttpServer.bind(
    "0.0.0.0",
    4040,
  );
  // #enddocregion bind
  print("Listening on localhost:${server.port}");

  // #docregion listen
  await for (HttpRequest request in server) {
    request.response.write("Hello World!");
    await request.response.close();
  }
  // #enddocregion listen
}
' > main.dart
```

Every request sent to port 4040 will receive a "Hello World!" response.

**4.** **\(Optional\)** **Test out the API.** [Install Dart](https://dart.dev/get-dart) and run the server: 

```text
$ dart main.dart
```

Then curl the API. You should receive the "Hello World!" response if the server is running normally:

```text
$curl localhost:4040
Hello World!
```

**5. Write a Dockerfile**. Similarly, directly paste the below command block into your terminal and you will have a `Dockerfile` created.

```text
$ echo 'FROM google/dart

ENV TINI_VERSION v0.18.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]

COPY . .
RUN dart2native main.dart -o a.out
CMD ["./a.out"]
'
```

A `Dockerfile` describes how a private filesystem of a container should be assembled and carry instructions on how a container is run based on the image built from it. Think of it as a step-by-step recipe on how an image is built, where the container instances can be started based on the image. 

The `Dockerfile` we have here describes how our dart server should be converted to an image.

A library called `tini` is included, to prevent spawns of zombie processes and provide default signal handlers in container instances.

At line 9, the dart server code `main.dart` is compiled to `a.out`. The latter file which functions exactly like the former is then executed at line 10. Containers deployed from the image built from this will have our dart server running on them.

Both the image building and container deploying processes are taken care of by Skygear, all you need to do is upload your `Dockerfile`. For more information on `Dockerfile`, please refer to its [official doc](https://docs.docker.com/engine/reference/builder/). 

**6. Write Skygear.yaml.** We are now one step away from deploying. Create a file called `skygear.yaml` so that your file tree in `dartserver` looks like this:

```text
.
└── dartserver
    ├── Dockerfile
    ├── main.dart
    └── skygear.yaml
```

Paste the following content into it. 

```text
app: <your_name>-dartserver      # your Skygear app name
deployments:
  server:                        # an arbitrary name of the service
    type: http-service           # type of the service
    context: .                   # root directory of your source code
    path: /                      # the path for the service
    port: 4040                   # the listening port of the service
```

The value given at line 1 is the name of app you have created on Skygear. This has nothing to do with the name of your local directory or program. 

Every item under `deployments`, i.e line 3-7 in our yaml file, is a service registered under an app. Files in our local directory `dartserver`, indicated as `.` since `skygear.yaml` resides in it, are uploaded and deployed as a service called `server`. Port 4040 where our Dart server responses with a "Hello World!" message is mapped to the service `server` under the path `/`.

**7. Deploy with one single command.** Be sure you are in the `dartserver` directory, as `skycli` will look for a `skygear.yaml` during deployment.

```bash
$ skycli app deploy
```

It's likely that you are going to see a list of logs. The deployment is finished when you receive the message "Deployment completed".

**8. Test out the app on Skygear.**

```text
curl https://<your_name>-dartserver.v2.dev.skygearapis.com/server
Hello World!%
```

\_\_TODO\_\_ update endpoint

##  Conclusion

We have now deployed a simple Dart server with our containerization instructions. No matter how intuitive or complex you application is, the same applies as you will just have to write a self-defined `Dockerfile`. Skygear will interpret the `Dockerfile` when supplied and finish up all the deployment chores for you.

## Useful tips

* To exclude deploying specific files, create a `.skyignore` file. It works the same as `.dockerignore`.

