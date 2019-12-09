# OAuth Authentication

### Login/Signup

User can login with OAuth provider. After authenticating at the provider site, user will be logged in as existing user, or signed up as new user if the user email is not yet registered.

Skygear will fetch user ID and profile from OAuth providers using provider-specific APIs. These information can be accessed through Skygear Auth API.

### Account Linkage

User can be linked with multiple OAuth provider account. In this case, an identity will be created for each linked provider account. At most one account for each provider can be linked to a user, and at most one user can be linked to a provider account. Users can log in using any of the linked provider account.

User can also unlink OAuth provider accounts. After unlinking, the account can be linked again with other users.

### Login/Signup with Access Token

User can log in using raw access token obtained through OAuth flow. This is commonly used when native SDK \(e.g. Facebook SDK\) integration is desired. However, this method has potential security risk, so this method is disabled by default. Developers should evaluate the security risk and decide whether to use this feature.

### References

* [Security risk of authentication through access token](https://oauth.net/articles/authentication/#access-tokens)

