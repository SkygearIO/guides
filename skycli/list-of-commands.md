# List of Commands

## Introduction

This document covers the commands supplied by `skycli`. Currently there are 5 groups of them:

* `app`: all commands under this are to interact or manage apps on Skygear
* `auth`: for authentication operations on `skycli`, like signing up
* `config`: for Skygear cluster operations on `skycli`
* `secret`: to manage secrets of a Skygear app
* `domain`: to manage custom domains of a Skygear app
* `version`: the only group where itself is a valid command, with no sub-commands

Type in `skygear --help` to print out these groups. To check the sub-commands, enter `skygear <group-name> --help`.

## Common Flags

#### `--app` 

When you are in some directories with no `skygear.yaml` inside, while you need to provide a Skygear app name to a `skycli` command, you can do this via the use of `--app`.

You won't need to do this when there exists a `skygear.yaml` in the current directory as `skycli` automatically looks for that and will parse the `app` value \(i.e. an app's name\) inside.

## `skycli app`

Whenever you need to interact with a Skygear app, go for these commands. Unless you are dealing with secrets, which should be done with the`skygear secret` commands. Most of them requires you to provide your app name.

#### `skycli app create`

Will ask you questions regarding the properties \(e.g. name\) of the app, then create one for you.

#### `skycli app list`

List all apps that you have access right.

#### `skycli app scaffold`

Kicks start a scaffolding for a Skygear app from our templates.

#### `skycli app deploy`

Deploys a Skygear app to your connected cluster. **Requires** a `skygear.yaml` at the same level defining the deployment.

#### `skycli app view-config`

Output your app's metadata and secrets.

#### `skycli app update-config`

Allow you to edit your app's metadata and secrets.

#### `skycli app add-collaborator [email]`

Add a collaborator to your app by sending them an email.

#### `skycli app remove-collaborator [email]`

Remove a collaborator with his/her email.

#### `skycli app list-collaborator`

List collaborator of your Skygear app.

#### `skycli app list-templates`

List out all templates of your Skygear app.

#### `skycli app update-templates`

Upload all valid templates in the folder `templates` a level under your `skygear.yaml`'s residence. 

#### `skycli app download-templates`

Download all templates of your Skygear app.

#### `skycli app get-k8s-credentials`

Write your Skygear app's Kubernetes credentials to your local config, normally at `~/.kube/config`.

#### `skycli app get-k8s-token-request`

Print out your app's `ExecCredential`.

#### `skycli app delete-k8s-credentials deploy`

Remove your app's  Kubernetes credentials from your local config.

## `skycli auth`

All authentication operations on `skycli` are done by these commands.

#### `skycli auth login`

Log in with email to use privileged commands of `skycli`.

#### `skycli auth logout`

Clean the account state of `skycli`.

#### `skycli auth signup`

Create a Skygear account in connected cluster. After you have provided the credentials, you will have to verify your email before you can use the account

## `skycli config`

#### `skycli config view`

Print out the metadata of the connected cluster.

#### `skycli config set-cluster`

Connect to a Skygear cluster by answering a set of questions.

## `skycli secret`

All about secret management under a Skygear app. Ensure app name is provided!

#### `skycli secret list`

List out all secrets under the specified app.

#### `skycli secret create [name] [value]`

Create a secret with the given name-value pair.

#### `skycli secret delete [name]`

By name, delete an existing secret.

## `skycli domain`

While Skygear gives your app a default domain with the suffix `.skygearapp.com`, you can replace this with a custom domain \(that you own\).

#### `skycli domain add [domain]`

Register a custom domain to your Skygear app. Lines of DNS records will be outputted after executing the command, be sure to add them to your DNS provider. Then, verify the domain to complete the flow.

#### `skycli domain list`

List all added custom domain of your Skygear app.

#### `skycli domain view [domain]`

Display the metadata of a registered custom domain.

#### `skycli app verify [domain]`

Verify an added custom domain.

#### `skycli app update [domain]`

Update verified custom domain.

#### `skycli app remove [domain]`

Remove a custom domain from your Skygear app.

## `skycli version`

No sub-commands, directly run this to print out your `skycli` version.

