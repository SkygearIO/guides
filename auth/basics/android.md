---
title: User Authentication Basics
description: User sign up, login, logout / access token / username, email and password management in Android
---

[[toc]]

Skygear support user authentication with email or username. The following
shows how it works using Skygear SDK.


## Managing User

### Signing up

The following shows how to sign up a user with username and password.

```java
Container skygear = Container.defaultContainer(this);

String username = getUsername(); // get from user input
String password = getPassword(); // get from user input

skygear.signupWithUsername(username, password, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Signup", "onAuthSuccess: Got token: " + user.accessToken);
    }

    @Override
    public void onAuthFail(String reason) {
        Log.i("Skygear Signup", "onAuthFail: Fail with reason: " + reason);
    }
});
```

Besides that, sign up with email is also supported:

```java
Container skygear = Container.defaultContainer(this);

String email = getEmail(); // get from user input
String password = getPassword(); // get from user input

skygear.signupWithEmail(email, password, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Signup", "onAuthSuccess: Got token: " + user.accessToken);
    }

    @Override
    public void onAuthFail(String reason) {
        Log.i("Skygear Signup", "onAuthFail: Fail with reason: " + reason);
    }
});
```

### Logging in

The following shows how to log in a user with username and password.

```java
Container skygear = Container.defaultContainer(this);

String username = getUsername(); // get from user input
String password = getPassword(); // get from user input

skygear.loginWithUsername(username, password, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Login", "onAuthSuccess: Got token: " + user.accessToken);
    }

    @Override
    public void onAuthFail(String reason) {
        Log.i("Skygear Login", "onAuthFail: Fail with reason: " + reason);
    }
});
```

Users can log in with their emails if they have registered:

```java
Container skygear = Container.defaultContainer(this);

String email = getEmail(); // get from user input
String password = getPassword(); // get from user input

skygear.loginWithUsername(email, password, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Login", "onAuthSuccess: Got token: " + user.accessToken);
    }

    @Override
    public void onAuthFail(String reason) {
        Log.i("Skygear Login", "onAuthFail: Fail with reason: " + reason);
    }
});
```


## Current User

After sign up / log in, the user session can be obtained by:

```java
User currentUser = skygear.getCurrentUser();

Log.i("Skygear User", "User access token: " + currentUser.accessToken);
Log.i("Skygear User", "User ID: " + currentUser.userId);
Log.i("Skygear User", "Username: " + currentUser.username);
```

Please be reminded that the `currentUser` object persist locally, and the
information (e.g. roles, emails, etc) might not sync with the server if it was
changed remotely.

To get the latest information of the current user, you can call `whoami()`.

```java
skygear.whoami(new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear User", "I am " + user.getUsername());
    }

    @Override
    public void onAuthFail(String reason) {
        // Error handling...
    }
});
```
