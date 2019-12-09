---
description: >-
  You can connect to your self-managed database to Skygear using secret
  management.
---

# How to connect to a managed database in Skygear

Most app requires a database. Although Skygear does not come with a database, you can bring in your own. 

**1. Create a secret** with the database connection information using `skycli`. You can also add and manage secrets at the Developer Portal.

```bash
$ skycli secret create <secret_name> '<database_url>'

#example
$ skycli secret create MONGO_DB_URL 'mongodb+srv://<USER>:<PASSWORD>@<HOST>/<COL>?retryWrites=true'
```

**2. Check if the secret has been created successfully.**

```text
$ skycli secret list
NAME           CREATED_AT                  
MONGO_DB_URL   2019-10-10 14:03:40 +08:00  
```

**3. Update `skygear.yaml`** in order to access the database secret in a service.

```yaml
  backend:
    type: http-service
    context: backend
    template: nodejs:12
    path: /api
    port: 8080
    secrets:
      - MONGO_DB_URL
```

**4. Deploy the Skygear app** to apply the changes.

```text
$ skycli app deploy
```

**5. Access the database secret in the service as an environment variable.**

```text
const url = process.env.MONGO_DB_URL;
```



