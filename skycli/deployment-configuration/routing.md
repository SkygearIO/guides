# Routing

Developers can specifies the routing configuration of the app. The routing behavior is based on simple rules:

* The request path is matched against configured `path` prefix.
* The longest matched prefix would be used.
* Path with `/_`prefix is reserved for Skygear built-in APIs.

For example, in the following configuration:

```yaml
deployments:
  - name: backend
    path: /api
    port: 8080
    # ...
  - name: frontend
    path: /
    port: 8080
    # ...
```

Requests would be routed to:

* `/`: micro-service `frontend`
* `/api`: micro-service `backend`
* `/login`: micro-service `frontend`
* `/api/blogs`: micro-service `backend`
* `/_auth/login`: Skygear Auth APIs

Requests with unmatched paths would be responded with status code 404.

