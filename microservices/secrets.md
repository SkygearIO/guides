# Secrets

Micro-services may requires some sensitive information at runtime, such as database credentials, encryption keys. Skygear allows developers to setup **Secrets** to provide sensitive information to micro-services securely.

Developers can specify the name and value of secret when creating the secret. Developers can also delete and list secrets. However, once the secret is created, the value cannot be retrieved through APIs.

When deploying micro-services, developer can specify which secrets should be injected into the execution environment. Once deployed, the specified secrets can be read in micro-services as environment variable. The name of environment variable is same as the secret name. Even if the secret is deleted, the existing deployed micro-services would not be affected.

