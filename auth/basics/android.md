---
title: User Authentication Basics
description: User sign up, login, logout / access token / username, email and password management in Android
---

[[toc]]

## Overview

Skygear supports user authentication with email or username. This guide will show you how it works using the Skygear Android SDK.

The global `Container` named `skygear` will be used throughout the examples in this guide:
```java
Container skygear = Container.defaultContainer(this); // Global Container
```

**Some basic concepts**

### Access token

Skygear handles the user login session using an access token stored in the local storage.

Each user, when logged in, will be given a generated String called `Access Token`, which is like a identification card for the user. It is used by the Skygear server to identify who you are.

The method `whoami` illustrates the usage of `Access Token`. When it is called, Skygear will send the current `Access Token` to the server and the server will return a `User` object.

### Database operation

The authenticated states of a user will affect the database operation he can perform.

Records in the public database can be queried without any user session. But if you want to write and/or update any records, a user session is required.

## Signing up

There are 2 APIs you can use to sign up a user:
1. `signupWithUsername`: signing up with a username and a password
2. `signupWithEmail`: signing up with an email and a password

The following example shows how to sign up a user with username and password.

```java
String username = getUsername(); // get from user input
String password = getPassword(); // get from user input

skygear.getAuth().signupWithUsername(username, password, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Signup", "onAuthSuccess: Got token: " + user.getAccessToken());
    }

    @Override
    public void onAuthFail(Error error) {
        Log.i("Skygear Signup", "onAuthFail: Fail with reason: " + error.getMessage());
    }
});
```

Signing up with email is done with much the same way:

```java
String email = getEmail(); // get from user input
String password = getPassword(); // get from user input

skygear.getAuth().signupWithEmail(email, password, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Signup", "onAuthSuccess: Got token: " + user.getAccessToken());
    }

    @Override
    public void onAuthFail(Error error) {
        Log.i("Skygear Signup", "onAuthFail: Fail with reason: " + error.getMessage());
    }
});
```

If a user try to sign up with an existing username / email, the error `ERROR.CODE.DUPLICATED` will be returned in the `onAuthFail` method.

A typical way to check if a user account already exists is shown below:

```java
skygear.getAuth().signupWithUsername(username, password, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Signup", "onAuthSuccess: Got token: " + user.getAccessToken());
    }

    @Override
    public void onAuthFail(Error error) {
        if(error.getCode().equals(Error.Code.DUPLICATED)){
            Toast.makeText(getApplicationContext(), "User already exist!", Toast.LENGTH_SHORT).show();
            //Duplicate user handling...
        }
    }
});
```

## Anonymous user

As mentioned before, to write and/or update a record in the Skygear database, there must be a user session.

However, if your app dose not require a user to sign up explicitly, how can you write records to the cloud database from your app then?

In this case you can create a anonymous user session. An anonymous user acts like a normal user - he can write, read and update the records.

To create an anonymous user, call `skygear.signupAnonymously()` as shown below:

```java
skygear.getAuth().signupAninymously(new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Signup", "onAuthSuccess: Got token: " + user.getAccessToken());
    }

    @Override
    public void onAuthFail(Error error) {
        Log.i("Skygear Signup", "onAuthFail: Fail with reason: " + error.getMessage());
    }
});
```

Each anonymous user has a unique ID that behaves exactly the same way as normal. As expected, an anonymous user does not have username, email nor password.

Because of this behavior, once the token of an anonymous user is lost, the account will not be able to access again. The records, however, will still persist.

## Logging in

Logging in behaves the same way as signing up a user.

Depending whether they sign up with email or username, you can log a user in with:

- loginWithUsername
- loginWithEmail

However, if user try to sign up with an existing username / email, the error `ERROR.CODE.DUPLICATED` will be returned in the `onAuthFail` method, indicating the username / email is already registered.

The following shows how to log in a user with username and password.

```java
String username = getUsername(); // get from user input
String password = getPassword(); // get from user input

skygear.getAuth().loginWithUsername(username, password, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Login", "onAuthSuccess: Got token: " + user.getAccessToken());
    }

    @Override
    public void onAuthFail(Error error) {
        Log.i("Skygear Login", "onAuthFail: Fail with reason: " + error.getMessage());
    }
});
```

Users can log in with their emails if they have registered:

```java
String email = getEmail(); // get from user input
String password = getPassword(); // get from user input

skygear.getAuth().loginWithUsername(email, password, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear Login", "onAuthSuccess: Got token: " + user.getAccessToken());
    }

    @Override
    public void onAuthFail(Error error) {
        Log.i("Skygear Login", "onAuthFail: Fail with reason: " + error.getMessage());
    }
});
```

## Logging out
To logout from the current user, simply call `skygear.logout()` as shown below:

```java
skygear.getAuth().logout(new LogoutResponseHandler() {
   @Override
   public void onLogoutSuccess() {
       Log.i("Skygear Logout", "Successfully logged out");
   }

   @Override
   public void onLogoutFail(Error error) {
        Log.i("Skygear Logout", "onLogoutFail: Fail with reason:" + error.getMessage());
   }

});
```

## Getting the current User

After sign up / log in, the user session can be obtained with `getCurrentUser`.

```java
Container skygear = Container.defaultContainer(this);
User currentUser = skygear.getAuth().getCurrentUser();

Log.i("Skygear User", "User access token: " + currentUser.getAccessToken());
Log.i("Skygear User", "User ID: " + currentUser.getId());
Log.i("Skygear User", "Username: " + currentUser.getUsername());
```

Please be reminded that the `currentUser` object persists locally, and the
information (e.g. roles, emails, etc) might not sync with the server if it was
changed remotely.

To get the latest information of the current user, you can call `whoami()`.

```java
Container skygear = Container.defaultContainer(this);

skygear.getAuth().whoami(new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(User user) {
        Log.i("Skygear User", "I am " + user.getUsername());
    }

    @Override
    public void onAuthFail(Error error) {
        Log.i("Skygear User", "onAuthFail: Fail with reason:" + error.getMessage());
    }
});
```

## Updating the username / email of a user

To change a user's username and email, you can use the `skygear.saveUser()` method by providing the user ID and the new username and/or the new email.

Each user can only edit his/her information. Only users with the admin role can change information of other users.

This is how you can update the username/email of the current user:

```java
User currentUser = skygear.getCurrentUser();

User newUser = new User(currentUser.getId(),
                        currentUser.getAccessToken(),
                        "your-new-username",
                        "your-new-email@hello.com");

skygear.getAuth().saveUser(newUser, new UserSaveResponseHandler() {
    @Override
    public void onSaveSuccess(User user) {
        Log.i("Skygear User", "Update user successful");
    }

    @Override
    public void onSaveFail(Error error) {
        Log.i("Skygear User", "onSaveFail: Fail with reason:" + error.getMessage());
    }
});
```

If you only want to change one of the user fields, just replace the new username/email with `currentUser.getUsername()`/`currentUser.getEmail()`.

## Updating the password of a user
Coming Soon

## What's next from here?

You may want to learn more about:
- [Social Login using Skygear (Coming soon)](https://docs.skygear.io/guides/auth/social-login/android/)
- [Setting User Profile (Coming soon)](https://docs.skygear.io/guides/auth/user-profile/android/)
