---
description: The deployment configuration file
---

# skygear.yaml

`skygear.yaml` defines the deployment configuration. It contains configuration for an app deployment:

* Micro-service configuration
* Web-hook configuration

A sample deployment configuration file looks like this:

```yaml
app: my-app

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

## deployments

The top-level key `deployments` contains a list of deployment items. In the above sample, there are two micro-service deployment item, named `backend` and `frontend`.

### name

The `name` key specifies the name of the deployment item. It must be unique within the list.

### type

The `type` key specifies the type of the deployment item. For now only [http-service](./#http-service) is supported. Each type of deployment item has its own additional properties.

### path

The `path` key represents the path which the deployment item would be mounted at. For details, refer to [Routing](routing.md) documentation.

## http-service

### port

The `port` key specifies the TCP port the micro-service is listening for. Required.

### command

The `command` key specifies the command of micro-service. It will be translated to `args` key of Kubernetes deployment.

### template, context and image

The `template`, `context`, and `image` key specifies micro-service's Docker image source. For details, refer to [Deployment Image](image-building.md) documentation.

### environment

The `environment` key specifies the environment variables of the micro-services.

* For simple environment variables, it can be configured with literal `name` and `value`.
* For environment variables using secret as source, the `secret` key is used. `name` is optional and default to the name of secret.

## hooks

Web-hook configurations are put under `hooks` key. Each entry represents a web-hook handler.

### event

The `event` key specifies the event name that the web-hook handler would handle. The same event name can appears multiple times in the list. For list of event names, refer to [Web-hooks](../../auth/web-hooks.md) documentation.

### path

The `path` key specifies the location of web-hook handler. It can be an absolute URL \(e.g. `https://example.com`\), or a absolute path \(which would be resolved to absolute URL based on the app endpoint\).



