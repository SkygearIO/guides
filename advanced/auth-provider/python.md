---
title: Writing your own Authentication Provider
---

A common use case of a social app is to authenticate user with popular social network. Authentication Provider allows a plugin to authenticate user based on credentials from a third-party. The Authentication Provider respond to Skygear Server whether the credentials are accepted.

When authenticated, Skygear creates new user account for new user or fetches an existing user account for an existing user. Skygear distinguish user account by User ID, which your Authentication Provider has to generate based on the supplied credentials.

Here is a simple cloud function example to implement Facebook login for your app.

```python
import skygear
import skygear.providers
import facebook

@skygear.provides("auth", "com.facebook")
class FacebookProvider(skygear.providers.BaseAuthProvider):
    def login(self, auth_data):
        graph = facebook.GraphAPI(access_token=auth_data['access_token'])
        auth_data.update(graph.get_object(id='me'))
        return {"principal_id": auth_data['id'], "auth_data": auth_data}
```

To create an Authentication Provider, you have to declare a class that implements `skygear.providers.BaseAuthProvider`. In this example, you need`import skygear.providers` as well as `import facebook` to use the Facebook SDK.

The decorator `@skygear.provides("auth", "com.facebook")` decorates the class and register it to Skygear. The first argument denotes that this is an Authentication Provider, and the provider has the name `com.facebook`. The provider name can be arbitrary, but it should adopt the reverse DNS name of your company as a convention.

Your app needs to obtain an access token by calling Facebook SDK from the client side. If you use relevant Skygear framework, the Facebook authentication flow will be handled for you automatically. In this example, the Authentication Provider will validates access token with Facebook and return the user’s Facebook ID to Skygear.

Plugin can also return additional authentication data to Skygear. Doing this allows you to store additional data to User object.
