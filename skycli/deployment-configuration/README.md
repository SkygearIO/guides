---
description: The deployment configuration file
---

# skygear.yaml

All deployment configuration of an app is defined with `skygear.yaml`. Configurations can be divided into two types, microservice and web hooks respectively.

A sample deployment configuration file looks like this:

```yaml
app: my-app

api_version: v2.1

deployments:
  - name: backend
    type: http-service
    path: /api
    port: 8080
    context: backend
    template: nodejs:12
    environment:
      - name: SERVER_PORT
        value: "8080"
      - secret: DATABASE_URL
  - name: frontend
    type: http-service
    path: /
    port: 8080
    context: frontend
    template: nodejs:12

hooks:
  - path: /api/after_user_create
    event: after_user_create
```

Configurations of the microservices are declared under the key `deployments`, while those of hooks are under the key `hooks`.

## api\_version

The top-level key `api_version` declares the API version. Use the value `v2.1` for now.

## deployments

The top-level key `deployments` contains a list of deployment items. In the above sample, there are two micro-service deployment items, named `backend` and `frontend`.

### name

The name of a deployment item, must be unique in this list \(i.e. under the key `deployments`\).

### type

Defines the type of a deployment item, where only [http-service](./#http-service) is supported.

### path

Indicates the path which the deployment item would be mounted and be available at. For details, refer to [Routing](routing.md) documentation.

## http-service

### port

The `port` key specifies the TCP port the micro-service is listening for. Required.

### command

The `command` key specifies the command of a microservice. It will be translated to the `args` key of Kubernetes deployment.

### template, context and image

The `template`, `context`, and `image` keys specify a microservice item's Docker image source. For details, refer to [Deployment Image](image-building.md) documentation.

### environment

The `environment` key specifies the environment variables of a microservice item.

* For environment variables, it can be configured with a literal pair of `name` and `value`.
* For environment variables using a Kubernetes Secret as the source, use the `secret` key. `name` is optional, with use the Secret's name when not given.

## hooks

Web-hook configurations are put under the `hooks` key. Each entry represents a web-hook handler.

### event

The `event` key reflects the event name that the web-hook handler would handle. The same event name can appears multiple times in the list. For list of event names, refer to [Web-hooks](../../auth/web-hooks.md) documentation.

### path

The `path` key specifies the location of web-hook handler. It can be an absolute URL \(e.g. `https://example.com`\), or path \(which would be resolved to absolute URL based on the app endpoint\).



