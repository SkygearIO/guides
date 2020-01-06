# Login IDs

In Skygear Auth, instead of the traditional username and email, we have login IDs. This model is developed based on the [Tripartite Identity Pattern](http://habitatchronicles.com/2008/10/the-tripartite-identity-pattern/). Each login ID can be email, username, phone, or whatever allowed in app configuration.

Developers can configure what types of login ID is allowed in app configuration, for example:

| Login ID Key | Login ID Type | Login ID |
| :--- | :--- | :--- |
| email | email | john.doe@example.com |
| username | username | johndoe |
| device\_id | raw | 4A72107E-EB7B |

User can login with any of the associated login ID.

### Login ID Types

Login ID types specifies the meaning of the login ID, so that Skygear Auth can perform normalization & validation on them.

* email: email in format specified by RFC5332
* username: username in format \(TODO\)
* phone: phone in E.164 format
* raw: a raw string

### Change Login IDs

Login IDs associated with a user can be added, removed, or updated their associated login ID using APIs.

