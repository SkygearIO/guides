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
```kotlin
val skygear = Container.defaultContainer(this)
```

## Basic concept

### Access token

Skygear handles the user login session using an access token stored in the local storage.

Each user, when logged in, will be given a generated String called `Access Token`, which is like an identification card for the user. It is used by the Skygear server to identify who you are.

The method `whoami` illustrates the usage of `Access Token`. When it is called, Skygear will send the current `Access Token` to the server and the server will return a `User` object.

### Database operation

The authenticated states of a user will affect the database operation he can perform.

Records in the public database can be queried without any user session. But if you want to write and/or update any records, a user session is required.

## Signing up

### Signing up with email or username

A user can sign up using a username or an email, along with a password.
It is done using either [`signupWithUsername`] or
[`signupWithEmail`].

Skygear does not allow duplicated usernames or emails. Signing up with a
duplicated identifier will give the error `Duplicated`.

While each of the sign-up functions is resolved with a user record,
in most cases you need not deal with it because
you can access the currently logged-in user using [`getCurrentUser`].

[`signupWithUsername`] sample code:

```java
String username = getUsername(); // get from user input
String password = getPassword(); // get from user input

skygear.getAuth().signupWithUsername(username, password, new AuthResponseHandler() {
  @Override
  public void onAuthSuccess(Record user) {
    String accessToken = skygear.getAuth().getCurrentAccessToken();
    Log.i("Skygear Signup", "onAuthSuccess: Got token: " + accessToken);
  }

  @Override
  public void onAuthFail(Error error) {
    if (error.getCode() == Error.Code.DUPLICATED) {
      Log.e("Skygear Signup", "Duplicated Username");
    } else {
      Log.e("Skygear Signup", "onAuthFail: Fail with reason: " + error.getMessage());
    }
  }
});
```
```kotlin
val username: String = getUsername() // get from user input
val password: String = getPassword() // get from user input

skygear.auth.signupWithUsername(username, password, object : AuthResponseHandler() {
  override fun onAuthSuccess(user: Record) {
    val accessToken = skygear.auth.currentAccessToken
    Log.i("Skygear Signup", "onAuthSuccess: Got token: $accessToken")
  }

  override fun onAuthFail(error: Error) {
    when (error.code) {
      Error.Code.DUPLICATED -> Log.e("Skygear Signup", "Duplicated Username")
      else -> Log.e("Skygear Signup", "onAuthFail: Fail with reason: ${error.message}")
    }
  }
})
```

[`signupWithEmail`] sample code:

```java
String email = getEmail(); // get from user input
String password = getPassword(); // get from user input

skygear.getAuth().signupWithEmail(email, password, new AuthResponseHandler() {
  @Override
  public void onAuthSuccess(Record user) {
    String accessToken = skygear.getAuth().getCurrentAccessToken();
    Log.i("Skygear Signup", "onAuthSuccess: Got token: " + accessToken);
  }

  @Override
  public void onAuthFail(Error error) {
    if (error.getCode() == Error.Code.DUPLICATED) {
      Log.e("Skygear Signup", "Duplicated Email");
    } else {
      Log.e("Skygear Signup", "onAuthFail: Fail with reason: " + error.getMessage());
    }
  }
});
```
```kotlin
val email: String = getEmail() // get from user input
val password: String = getPassword() // get from user input

skygear.auth.signupWithEmail(email, password, object : AuthResponseHandler() {
  override fun onAuthSuccess(user: Record) {
    val accessToken = skygear.auth.currentAccessToken
    Log.i("Skygear Signup", "onAuthSuccess: Got token: $accessToken")
  }

  override fun onAuthFail(error: Error) {
    when (error.code) {
      Error.Code.DUPLICATED -> Log.e("Skygear Signup", "Duplicated Email")
      else -> Log.e("Skygear Signup", "onAuthFail: Fail with reason: ${error.message}")
    }
  }
})
```

It is common to add other data to user record when signing up, you can do that
by specifying the data for user record in the third parameter of the signup
functions:

```java
String username = getUsername(); // get from user input
String password = getPassword(); // get from user input
Map<String, Object> profile = new HashMap<String, Object>();
profile.put("interest", "reading");

skygear.getAuth().signupWithUsername(username, password, profile, new AuthResponseHandler() {
  @Override
  public void onAuthSuccess(Record user) {
    String accessToken = skygear.getAuth().getCurrentAccessToken();
    Log.i("Skygear Signup", "onAuthSuccess: Got token: " + accessToken);
    Log.i("Skygear Signup", "interest: " + user.get("interest"));
  }

  @Override
  public void onAuthFail(Error error) {
    if(error.getCode() == Error.Code.DUPLICATED) {
      Log.e("Skygear Signup", "Duplicated Username");
    } else {
      Log.e("Skygear Signup", "onAuthFail: Fail with reason: " + error.getMessage());
    }
  }
});
```
```kotlin
val username: String = getUsername() // get from user input
val password: String = getPassword() // get from user input
val profile = HashMap<String, Any>()
profile.put("interest", "reading")

skygear.auth.signupWithUsername(username, password, profile, object : AuthResponseHandler() {
  override fun onAuthSuccess(user: Record) {
    val accessToken = skygear.auth.currentAccessToken
    Log.i("Skygear Signup", "onAuthSuccess: Got token: $accessToken")
    Log.i("Skygear Signup", "interest: ${user.get("interest")}")
  }

  override fun onAuthFail(error: Error) {
    when (error.code) {
      Error.Code.DUPLICATED -> Log.e("Skygear Signup", "Duplicated Username")
      else -> Log.e("Skygear Signup", "onAuthFail: Fail with reason: ${error.message}")
    }
  }
})
```

### Signing up anonymously

Without being authenticated, a user can read data from the public database but
cannot perform most of the other operations, including saving data into the
database.

If you need an authenticated user but do not require a user to
sign up explicitly with a username or email, you can create an anonymous user
by calling [`signupAnonymously`].

Every anonymous user has a unique user ID, and behaves exactly the same as
any user authenticated with a username or an email. The only difference is that
an anonymous user has no username, email, nor password. Because of the absence
of username and email, the account will be lost when the access token is lost.

```java
skygear.getAuth().signupAnonymously(new AuthResponseHandler() {
  @Override
  public void onAuthSuccess(Record user) {
    String accessToken = skygear.getAuth().getCurrentAccessToken();
    Log.i("Skygear Signup", "onAuthSuccess: Got token: " + accessToken);
  }

  @Override
  public void onAuthFail(Error error) {
    Log.e("Skygear Signup", "onAuthFail: Fail with reason: " + error.getMessage());
  }
});
```
```kotlin
skygear.auth.signupAnonymously(object : AuthResponseHandler() {
  override fun onAuthSuccess(user: Record) {
    val accessToken = skygear.auth.currentAccessToken
    Log.i("Skygear Signup", "onAuthSuccess: Got token: $accessToken")
  }

  override fun onAuthFail(error: Error) {
    Log.e("Skygear Signup", "onAuthFail: Fail with reason: ${error.message}")
  }
})
```

## Logging in

The login functions are similar to the sign-up ones.

If the credentials are incorrect, it will give the error of:

- `InvalidCredentials` if the password is incorrect;
- `ResourceNotFound` if the email or username is not found.

While each of the login functions is resolved with a user record,
in most cases you need not deal with it because
you can access the currently logged-in user using [`getCurrentUser`].

### Logging in using a username

```java
String username = getUsername(); // get from user input
String password = getPassword(); // get from user input

skygear.getAuth().loginWithUsername(username, password, new AuthResponseHandler() {
  @Override
  public void onAuthSuccess(Record user) {
    String accessToken = skygear.getAuth().getCurrentAccessToken();
    Log.i("Skygear Login", "onAuthSuccess: Got token: " + accessToken);
  }

  @Override
  public void onAuthFail(Error error) {
    if (error.getCode() == Error.Code.INVALID_CREDENTIALS) {
      Log.e("Skygear Login", "Password incorrect");
    } else if (error.getCode() == Error.Code.RESOURCE_NOT_FOUND) {
      Log.e("Skygear Login", "No such username");
    } else {
      Log.e("Skygear Login", "onAuthFail: Fail with reason: " + error.getCode());
    }
  }
});
```
```kotlin
val username: String = getUsername() // get from user input
val password: String = getPassword() // get from user input

skygear.auth.loginWithUsername(username, password, object : AuthResponseHandler() {
  override fun onAuthSuccess(user: Record) {
    val accessToken = skygear.auth.currentAccessToken
    Log.i("Skygear Login", "onAuthSuccess: Got token: $accessToken")
  }

  override fun onAuthFail(error: Error) {
    when (error.code) {
      Error.Code.INVALID_CREDENTIALS -> Log.e("Skygear Login", "Password incorrect")
      Error.Code.RESOURCE_NOT_FOUND -> Log.e("Skygear Login", "No such username")
      else -> Log.e("Skygear Login", "onAuthFail: Fail with reason: ${error.code}")
    }
  }
})
```

### Logging in using an email

```java
String email = getEmail(); // get from user input
String password = getPassword(); // get from user input

skygear.getAuth().loginWithEmail(email, password, new AuthResponseHandler() {
  @Override
  public void onAuthSuccess(Record user) {
    String accessToken = skygear.getAuth().getCurrentAccessToken();
    Log.i("Skygear Login", "onAuthSuccess: Got token: " + accessToken);
  }

  @Override
  public void onAuthFail(Error error) {
    if (error.getCode() == Error.Code.INVALID_CREDENTIALS) {
      Log.e("Skygear Login", "Password incorrect");
    } else if (error.getCode() == Error.Code.RESOURCE_NOT_FOUND) {
      Log.e("Skygear Login", "No such email");
    } else {
      Log.e("Skygear Login", "onAuthFail: Fail with reason: " + error.getCode());
    }
  }
});
```
```kotlin
val email: String = getEmail() // get from user input
val password: String = getPassword() // get from user input

skygear.auth.loginWithEmail(email, password, object : AuthResponseHandler() {
  override fun onAuthSuccess(user: Record) {
    val accessToken = skygear.auth.currentAccessToken
    Log.i("Skygear Login", "onAuthSuccess: Got token: $accessToken")
  }

  override fun onAuthFail(error: Error) {
    when (error.code) {
      Error.Code.INVALID_CREDENTIALS -> Log.e("Skygear Login", "Password incorrect")
      Error.Code.RESOURCE_NOT_FOUND -> Log.e("Skygear Login", "No such email")
      else -> Log.e("Skygear Login", "onAuthFail: Fail with reason: ${error.code}")
    }
  }
})
```

## Logging out

Logging out the current user is simple using the [`logout`] method.

Upon successful logout, the SDK will clear the current user and the access token
from the local storage.

```java
skygear.getAuth().logout(new LogoutResponseHandler() {
  @Override
  public void onLogoutSuccess() {
     Log.i("Skygear Logout", "Successfully logged out");
  }

  @Override
  public void onLogoutFail(Error error) {
      Log.e("Skygear Logout", "onLogoutFail: Fail with reason:" + error.getMessage());
  }
});
```
```kotlin
skygear.auth.logout(object : LogoutResponseHandler() {
  override fun onLogoutSuccess() {
    Log.i("Skygear Logout", "Successfully logged out")
  }

  override fun onLogoutFail(error: Error) {
    Log.e("Skygear Logout", "onLogoutFail: Fail with reason: ${error.message}")
  }
})
```

## Getting the current User

You can retrieve the current user from [`getCurrentUser`]. Please make sure the
current user has already signed up and had a username specified.

```java
Container skygear = Container.defaultContainer(this);
Record currentUser = skygear.getAuth().getCurrentUser();
String accessToken = skygear.getAuth().getCurrentAccessToken();

Log.i("Skygear User", "User access token: " + accessToken);
Log.i("Skygear User", "User ID: " + currentUser.getId());
Log.i("Skygear User", "Username: " + currentUser.get("username"));
```
```kotlin
val skygear = Container.defaultContainer(this)
val currentUser = skygear.auth.currentUser
val accessToken = skygear.auth.currentAccessToken

Log.i("Skygear User", "User access token: $accessToken")
Log.i("Skygear User", "User ID: " + currentUser.id)
Log.i("Skygear User", "Username: " + currentUser.get("username"))
```

If there is an authenticated user, it will give you a [`Record`] which
is the user record. The user record looks like this:

``` json
{
  '_id': 'abcdef',
  'username': 'Ben',
  'email': 'ben@skygeario.com',
}
```

Please be reminded that the [`getCurrentUser`]
 object persist locally, and the data in the user record
 might not sync with the server if it was changed remotely.

To get the latest information of the current user,
you can call [`whoami`]:

```java
Container skygear = Container.defaultContainer(this);

skygear.getAuth().whoami(new AuthResponseHandler() {
  @Override
  public void onAuthSuccess(Record user) {
    Object username = user.get("username");
    if (username != null) {
      Log.i("Skygear User", "I am " + username);
    } else {
      Log.w("Skygear User", "The user doesn't have a username!");
    }
  }

  @Override
  public void onAuthFail(Error error) {
    Log.e("Skygear User", "onAuthFail: Fail with reason:" + error.getMessage());
  }
});
```
```kotlin
val skygear = Container.defaultContainer(this)

skygear.auth.whoami(object : AuthResponseHandler() {
  override fun onAuthSuccess(user: Record) {
    val username = user.get("username")
    if (username != null) {
      Log.i("Skygear User", "I am $username")
    } else {
      Log.w("Skygear User", "The user doesn't have a username!")
    }
  }

  override fun onAuthFail(error: Error) {
    Log.e("Skygear User", "onAuthFail: Fail with reason: ${error.message}")
  }
})
```

## Updating a user's username and email

Username and email is saved to the user record. You can modify the username
and email in the same way you modify any other record, which is achieved by
calling [`save`].

::: caution

**Caution:** When saving user record, the user record returned by
[`getCurrentUser`] is not automatically updated. To force an update of
the record returned from [`getCurrentUser`], call [`whoami`].

:::

To change the username of the current user:

```java
final Container skygear = Container.defaultContainer(this);
Record currentUser = skygear.getAuth().getCurrentUser();
currentUser.set("username", "new-username");
skygear.getPublicDatabase().save(currentUser, new RecordSaveResponseHandler() {
  @Override
  public void onSaveSuccess(Record[] records) {
    for (int i = 0; i < records.length; i++) {
      Record user = records[i];
      Log.i("Skygear User", "Username: " + user.get("username"));
      skygear.getAuth().whoami(null);
    }
  }

  @Override
  public void onPartiallySaveSuccess(Map<String, Record> successRecords, Map<String, Error> errors) {
    for (Map.Entry<String, Record> entry : successRecords.entrySet()) {
      String recordId = entry.getKey();
      String username = entry.getValue().get("username").toString();

      Log.i("Skygear User", "Successfully saved - recordId: " + recordId + ", username: " + username);
    }

    for (Map.Entry<String, Error> entry: errors.entrySet()) {
      String recordId = entry.getKey();
      String errorMessage = entry.getValue().getMessage();

      Log.e("Skygear User", "Save error - recordId: " + recordId + ", reason: " + errorMessage);
    }
  }

  @Override
  public void onSaveFail(Error error) {
    Log.e("Skygear User", "onSaveFail: Fail with reason:" + error.getMessage());
  }
});
```
```kotlin
val skygear = Container.defaultContainer(this)
val currentUser = skygear.auth.currentUser
currentUser.set("username", "new-username")
skygear.publicDatabase.save(currentUser, object : RecordSaveResponseHandler() {
  override fun onSaveSuccess(users: Array<out Record>) {
    users.forEach { user ->
      Log.i("Skygear User", "Username: ${user.get("username")}")
      skygear.auth.whoami(null)
    }
  }

  override fun onPartiallySaveSuccess(successRecords: MutableMap<String, Record>, errors: MutableMap<String, Error>) {
    successRecords.forEach { (recordId, record) ->
      Log.i("Skygear User", "Successfully saved - recordId: $recordId, username: ${record.get("username")}")
    }

    errors.forEach { (recordId, error) ->
      Log.e("Skygear User", "Save error - recordId: $recordId, reason: ${error.message}")
    }
  }


  override fun onSaveFail(error: Error) {
    Log.e("Skygear User", "onSaveFail: Fail with reason: ${error.message}")
  }
})
```

To change the email of the current user:

```java
final Container skygear = Container.defaultContainer(this);
Record currentUser = skygear.getAuth().getCurrentUser();
currentUser.set("email", "new-email");
skygear.getPublicDatabase().save(currentUser, new RecordSaveResponseHandler() {
  @Override
  public void onSaveSuccess(Record[] records) {
    for (int i = 0; i < records.length; i++) {
      Record user = records[i];
      Log.i("Skygear User", "Email: " + user.get("email"));
      skygear.getAuth().whoami(null);
    }
  }

  @Override
  public void onPartiallySaveSuccess(Map<String, Record> successRecords, Map<String, Error> errors) {
    for (Map.Entry<String, Record> entry : successRecords.entrySet()) {
      String recordId = entry.getKey();
      String email = entry.getValue().get("email").toString();

      Log.i("Skygear User", "Successfully saved - recordId: " + recordId + ", email: " + email);
    }

    for (Map.Entry<String, Error> entry: errors.entrySet()) {
      String recordId = entry.getKey();
      String errorMessage = entry.getValue().getMessage();

      Log.e("Skygear User", "Save error - recordId: " + recordId + ", reason: " + errorMessage);
    }
  }

  @Override
  public void onSaveFail(Error error) {
    Log.e("Skygear User", "onSaveFail: Fail with reason:" + error.getMessage());
  }
});
```
```kotlin
val skygear = Container.defaultContainer(this)
val currentUser = skygear.auth.currentUser
currentUser.set("email", "new-email")
skygear.publicDatabase.save(currentUser, object : RecordSaveResponseHandler() {
  override fun onSaveSuccess(users: Array<out Record>) {
    users.forEach { user ->
      Log.i("Skygear User", "Email: ${user.get("email")}")
      skygear.auth.whoami(null)
    }
  }

  override fun onPartiallySaveSuccess(successRecords: MutableMap<String, Record>, errors: MutableMap<String, Error>) {
    successRecords.forEach { (recordId, record) ->
      Log.i("Skygear User", "Successfully saved - recordId: $recordId, email: ${record.get("email")}")
    }

    errors.forEach { (recordId, error) ->
      Log.e("Skygear User", "Save error - recordId: $recordId, reason: ${error.message}")
    }
  }

  override fun onSaveFail(error: Error) {
    Log.e("Skygear User", "onSaveFail: Fail with reason: ${error.message}")
  }
})
```

You can even change the username and email at the same time:

```java
final Container skygear = Container.defaultContainer(this);
Record currentUser = skygear.getAuth().getCurrentUser();
currentUser.set("username", "new-username");
currentUser.set("email", "new-email");
skygear.getPublicDatabase().save(currentUser, new RecordSaveResponseHandler() {
  @Override
  public void onSaveSuccess(Record[] records) {
    for (int i = 0; i < records.length; i++) {
      Record user = records[i];
      Log.i("Skygear User", "Username: " + user.get("username"));
      Log.i("Skygear User", "Email: " + user.get("email"));
      skygear.getAuth().whoami(null);
    }
  }

  @Override
  public void onPartiallySaveSuccess(Map<String, Record> successRecords, Map<String, Error> errors) {
    for (Map.Entry<String, Record> entry : successRecords.entrySet()) {
      String recordId = entry.getKey();
      String username = entry.getValue().get("username").toString();
      String email = entry.getValue().get("email").toString();

      Log.i("Skygear User", "Successfully saved - " +
        "recordId: " + recordId +
        ", username: " + username +
        ", email: " + email
      );
    }

    for (Map.Entry<String, Error> entry: errors.entrySet()) {
      String recordId = entry.getKey();
      String errorMessage = entry.getValue().getMessage();

      Log.e("Skygear User", "Save error - recordId: " + recordId + ", reason: " + errorMessage);
    }
  }


  @Override
  public void onSaveFail(Error error) {
    Log.e("Skygear User", "onSaveFail: Fail with reason:" + error.getMessage());
  }
});
```
```kotlin
val skygear = Container.defaultContainer(this)
val currentUser = skygear.auth.currentUser
currentUser.set("username", "new-username")
currentUser.set("email", "new-email")
skygear.publicDatabase.save(currentUser, object : RecordSaveResponseHandler() {
  override fun onSaveSuccess(users: Array<out Record>) {
    users.forEach { user ->
      Log.i("Skygear User", "Username: ${user.get("username")}")
      Log.i("Skygear User", "Email: ${user.get("email")}")
      skygear.auth.whoami(null)
    }
  }

  override fun onPartiallySaveSuccess(successRecords: MutableMap<String, Record>, errors: MutableMap<String, Error>) {
    successRecords.forEach { (recordId, record) ->
      Log.i("Skygear User", "Successfully saved - recordId: $recordId, email: ${record.get("username")}, email: ${record.get("email")}")
    }

    errors.forEach { (recordId, error) ->
      Log.e("Skygear User", "Save error - recordId: $recordId, reason: ${error.message}")
    }
  }

  override fun onSaveFail(error: Error) {
    Log.e("Skygear User", "onSaveFail: Fail with reason: ${error.message}")
  }
})
```

## Updating a user's password

The currently logged-in user can change his/her own password.
This can be done using the [`changePassword`] method.

If the current password is incorrect, the SDK will return an
`INVALID_CREDENTIALS` error.

```java
Container skygear = Container.defaultContainer(this);
skygear.getAuth().changePassword("new-password", "old-password", new AuthResponseHandler() {
  @Override
  public void onAuthSuccess(Record user) {
      Log.i("Change Password", "onAuthSuccess: Changed password successfully.");
  }

  @Override
  public void onAuthFail(Error error) {
      Log.e("Change Password", "onAuthFail: Fail with reason: " + error.getMessage());
  }
})
```
```kotlin
val skygear = Container.defaultContainer(this)
skygear.auth.changePassword("new-password", "old-password", object : AuthResponseHandler() {
  override fun onAuthSuccess(user: Record) {
    Log.i("Change Password", "onAuthSuccess: Changed password successfully.")
  }

  override fun onAuthFail(error: Error) {
    Log.e("Change Password", "onAuthFail: Fail with reason: ${error.message}")
  }
})
```

## Forgot password

Coming soon.

## What's next from here?

You may want to learn more about:
- [Social Login using Skygear (Coming soon)](/guides/auth/social-login/android/)
- [Setting User Profile](/guides/auth/user-profile/android/)

[`Record`]: https://docs.skygear.io/android/reference/v1/io/skygear/skygear/Record.html
[`getCurrentUser`]: https://docs.skygear.io/android/reference/v1/io/skygear/skygear/AuthContainer.html#getCurrentUser--
[`logout`]: https://docs.skygear.io/android/reference/v1/io/skygear/skygear/AuthContainer.html#logout-io.skygear.skygear.LogoutResponseHandler-
[`signupAnonymously`]: https://docs.skygear.io/android/reference/v1/io/skygear/skygear/AuthContainer.html#signupAnonymously-io.skygear.skygear.AuthResponseHandler-
[`signupWithEmail`]: https://docs.skygear.io/android/reference/v1/io/skygear/skygear/AuthContainer.html#signupWithEmail-java.lang.String-java.lang.String-io.skygear.skygear.AuthResponseHandler-
[`signupWithUsername`]: https://docs.skygear.io/android/reference/v1/io/skygear/skygear/AuthContainer.html#signupWithUsername-java.lang.String-java.lang.String-io.skygear.skygear.AuthResponseHandler-
[`save`]: https://docs.skygear.io/android/reference/v1/io/skygear/skygear/Database.html#save-io.skygear.skygear.Record:A-io.skygear.skygear.RecordSaveResponseHandler-
[`whoami`]: https://docs.skygear.io/android/reference/v1/io/skygear/skygear/AuthContainer.html#whoami-io.skygear.skygear.AuthResponseHandler-
[`changePassword`]: https://docs.skygear.io/android/reference/latest/io/skygear/skygear/AuthContainer.html#changePassword-java.lang.String-java.lang.String-io.skygear.skygear.AuthResponseHandler-
