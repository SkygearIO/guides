# Deployment Configuration

The deployment configuration is a file `skygear.yaml`. It contains configuration for an app deployment:

* Micro-service configuration
* Web-hook configuration

A sample deployment configuration file looks like this:

```yaml
app: my-app

deployments:
  backend:
    type: http-service
    path: /api
    port: 8080
    context: backend
    template: nodejs:12
    environment:
      - name: SERVER_PORT
        value: "8080"
      - secret: DATABASE_URL
  frontend:
    type: http-service
    path: /
    port: 8080
    context: frontend
    template: nodejs:12

hooks:
  - path: /api/after_user_create
    event: after_user_create
```

### Micro-service configuration

Micro-service configurations are put under `deployments` key. Each entry represents a micro-service deployment. In the above sample, there are two micro-service deployment, named `backend` and `frontend`.

The `type` key specifies the type of micro-service. For now only `http-service` is supported.

The `path` key represents the path which the micro-service would be mounted at. For details, refer to [Routing](routing.md) documentation.

The `port` key represents the port which the micro-service server is listening to.

The `template`, `context`, and `image` key specifies micro-service's Docker image source. For details, refer to [Deployment Image](image-building.md) documentation.

The `command` key specifies the startup command of micro-service. It will be translated to `args` key of Kubernetes deployment.

The `environment` key specifies the environment variables of the micro-services:

* For simple environment variables, it can be configured with literal `name` and `value`.
* For environment variables using secret as source, the `secret` key is used. `name` is optional and default to the name of secret.

### Web-hook configuration

Web-hook configurations are put under `hooks` key. Each entry represents a web-hook handler.

The `event` key specifies the event name that the web-hook handler would handle. The same event name can appears multiple times in the list. For list of event names, refer to [Web-hooks](../../auth/web-hooks.md) documentation.

The `path` key specifies the location of web-hook handler. It can be an absolute URL \(e.g. `https://example.com`\), or a relative path \(which would be resolved to absolute URL based on the app endpoint\).



