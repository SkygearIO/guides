# How to view the logs of your microservices

## Introduction

In a lot of cases a developer will need to look at the logs of a service to monitor health status, trace a bug or evaluate performance. This section walks you through how can you check the logging of a microservice deployed to Skygear be retrieved.

## Prerequisites

Have a Skygear account and an app registered. We will be using Secret on Skygear, which is a resource designed to hold sensitive information, registered under and can used by a Skygear app. The latter part of this definition means you will need an app before you can add Secrets.

Before you start this section, please ensure you have `skycli` installed and configured properly.

Although this section doesn't really require `skycli` as we will be adding database connection via Secrets which can be done with Skygear Portal, to try out the newly connected database you will still need an app deployed and `skycli` is the only way to perform that.

Also, make sure the version of your `kubectl` is &gt;= 1.15.

**TL**;**DR** Sooner or later you will need to install and configure `skycli`.

## Commands to retrieve logs

While you can access logs with `kubectl` by entering bunch of commands, commands from `skycli` have some of the details and operations abstracted.

This writes the context of your Skygear cluster to your local k8s configuration file:

```bash
$ skycli --app yourapp app get-k8s-credentials
# The necessary configuration was written to KUBECONFIG.
# Follow the instruction in the output to
# switch kubectl context if you wish.
```

Then you can verify the configuration was correctly set by `skycli`.

```bash
$ kubectl config view
```

If `current-context` of kubectl isn't that of your Skygear cluster, you can set it with:

```bash
$ kubectl config use-context <skygear_context_name>
```

Now the set-up is done, you can list out your pods \(i.e. the microservices\):

```bash
$ kubectl get pod
```

Then put in the name of the microservice you would like its logs to be retrieved:

```bash
$ kubectl logs <your_target pod>
```

