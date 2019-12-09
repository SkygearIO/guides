# User & Identity

Skygear Auth has two main concepts: **User** and **Identity**. Each user in your app may have multiple identity, while each identity will correspond to single user.

For example, a user John Doe may have three identity:

* Email of _john.doe@example.com_
* Username of _johndoe_
* Facebook account of _John Doe_

To authenticate in Skygear, the user need to have their identity verified. Skygear supports three type of identities: password-based, OAuth, and custom token.

### Password-based Identity

This identity type would be used when user signed up with a **Login ID** and **password**. User can then login with the same login ID and password.

By default, Skygear allows three types of login ID: email, username, and phone.

### OAuth Identity

This identity type would be used when user login with OAuth provider. Currently Skygear supports the following OAuth providers:

* Facebook
* Google
* Instagram
* LinkedIn
* Azure Active-Directory v2



