---
title: User Authentication Basics
---
With Skygear, you can implement a user login system easily using the
authentication functions provided in the SDK.

<a name="user-login-flow"></a>
## User Login Flow

Skygear stores and manages the user credentials (username, email and password)
internally and handles the user login session using an
[access token](/server/guide#others) stored in the local storage.

The following diagram shows the login flow:

<div style="text-align: center;">

```xml
    +-------------------------------------+    
    |        Initial page load and        |    
    |     Skygear container configured    |    
    +-------------------------------------+    

                      |                        
                      |                        
                      v                        

    +-------------------------------------+    
    |   1. Check if authenticated user    |    
    |      exists in local storage        |    
    +-------------------------------------+    

                      |                        
                      |                        
          +-----------+-----------+            
          |                       |            
   2. No  |                       |  3. Yes    
          |                       |            
          v                       v            

+-------------------+   +---------------------+
|  Anonymous state  |   |   Logged-in state   |
| (no access token) |   | (with access token) |
+-------------------+   +---------------------+

  ^   |                                 ^   |  
  |   |    4. login/sign-up success     |   |  
  |   +---------------------------------+   |  
  |                                         |  
  +-----------------------------------------+  
   5. logout success or access token expired   
```

</div>

1. When the container is configured with the server endpoint and the API key, it
   will check for the existence of an authenticated user in the local storage.
2. If there is none, the SDK will do nothing, the app remains in the anonymous
   state. There is no access token.
3. If there is, the SDK will retrieve the user information, including the
   user object (email, username, etc.) and the access token.
4. When a user logs in or signs up successfully, the SDK will set the user
   information with the access token obtained from the Skygear server. The app
   will change from the anonymous state to the logged-in state. The access token
   is sent to the server for authentication in every API call made through the
   SDK.
5. When a user logs out of the access token is found expired during an API
   call to the server, the SDK will clear the user information and the access
   token. The app will change from the logged-in state to the anonymous state.

<a name="user-login-status"></a>
## User Login Status

### Getting the currently logged-in user

You can retrieve the current user from `skygear.currentUser`.

``` javascript
const user = skygear.currentUser; // if not logged in, it will be null
```

If there is an authenticated user, it will give you a user object like this:

``` javascript
{
  'id': 'abcdef',
  'username': 'Ben',
  'email': 'ben@skygeario.com',
}
```

Please be reminded that the `currentUser` object only retrieved when user
login / signup Skygear.

To get the latest information (e.g. roles, emails, etc.) of the current user,
you can ask "Who am I" to Skygear:

``` javascript
skygear.whoami().then((user) => {
    console.log(`Oh. I am ${user.username}.`);
}, (err) => {
    // Error handling...
})

```

### Observing user changes

The preferred way for your app to handle any logged-in user change is to
register a callback by using the `skygear.onUserChanged` method.
The callback will be invoked whenever the user is changed, i.e.
when any of the followings happens:

1. a user logs in;
2. a user logs out (or as logged out due to an expired access token);
3. the current user changes his/her email, even if the new email is the
   same as the old one.

The callback will receive the new user object as the argument.

``` javascript
const handler = skygear.onUserChanged(function (user) {
  if (user) {
    console.log('user logged in or signed up');
  } else {
    console.log('user logged out or the access token expired');
  }
});

handler.cancel(); // The callback is cancelable
```


<a name="signup-login-logout"></a>
## Signing up / Logging in / Logging out

### Signing up

A user can sign up using a username or an email, along with a password.
It is done using either `skygear.signupWithUsername` or
`skygear.signupWithEmail`.

Skygear does not allow duplicated usernames or emails. Signing up with a
duplicated identifier will give the error `Duplicated`.

While each of the sign-up functions is resolved with a user object,
in most cases you need not deal with it because
you can access the currently logged-in user using `skygear.currentUser`.

#### Anonymous user

Without being authenticated, a user can read data from the public database but
cannot perform most of the other operations, including saving data into the
database.

If you need an authenticated user but do not require a user to
sign up explicitly with a username or email, you can create an anonymous user
by calling `skygear.signupAnonymously`.

Every anonymous user has a unique user ID, and behaves exactly the same as
any user authenticated with a username or an email. The only difference is that
an anonymous user has no username, email, nor password. Because of the absence
of username and email, the account will be lost when the access token is lost.

#### Signing up using a username

``` javascript
import skygear from 'skygear';
import skygearError from 'skygear/lib/error';

skygear.signupWithUsername(username, password).then((user) => {
  console.log(user); // user object
}, (error) => {
  console.error(error);
  if (error.error.code === skygearError.Duplicated) {
    // the username has already existed
  } else {
    // other kinds of error
  }
});
```

#### Signing up using an email

``` javascript
import skygear from 'skygear';
import skygearError from 'skygear/lib/error';

skygear.signupWithEmail(email, password).then((user) => {
  console.log(user); // user object
}, (error) => {
  console.error(error);
  if (error.error.code === skygearError.Duplicated) {
    // the email has already existed
  } else {
    // other kinds of error
  }
});
```

#### Signing up anonymously

``` javascript
import skygear from 'skygear';
import skygearError from 'skygear/lib/error';

skygear.signupAnonymously().then((user) => {
  console.log(user); // user object with undefined email and username
}, (error) => {
  console.error(error);
});
```

### Logging in

The login functions are similar to the sign-up ones.

If the credentials are incorrect, it will give the error of:

- `InvalidCredentials` if the password is incorrect;
- `ResourceNotFound` if the email or username is not found.

While each of the login functions is resolved with a user object,
in most cases you need not deal with it because
you can access the currently logged-in user using `skygear.currentUser`.

#### Logging in using a username

``` javascript
skygear.loginWithUsername(username, password).then((user) => {
  console.log(user); // user object
}, (error) => {
  console.error(error);    
  if (error.error.code === skygearError.InvalidCredentials ||
      error.error.code === skygearError.ResourceNotFound ) {
    // incorrect username or password
  } else {
    // other kinds of error
  }
})
```

#### Logging in using an email

``` javascript
skygear.loginWithEmail(email, password).then((user) => {
  console.log(user); // user object
}, (error) => {
  console.error(error);    
  if (error.error.code === skygearError.InvalidCredentials ||
      error.error.code === skygearError.ResourceNotFound ) {
    // incorrect username or password
  } else {
    // other kinds of error
  }
})
```

### Logging out

Logging out the current user is simple using the `skygear.logout` method.

Upon successful logout, the SDK will clear the current user and the access token
from the local storage.

``` javascript
skygear.logout().then(() => {
  console.log('logout successfully');
}, (error) => {
  console.error(error);
});
```

<a name="change-email-password"></a>
## Changing username/email/password

### Changing the username and email of a user

To change a user's username and email, you can use
the `skygear.saveUser` method by
providing the user ID and the new username and/or the new email.
Every user can change his/her own username/email while
only users with the admin role can change the usernames/emails of other users.

<div class="caution">

**Caution:** Changing the username/email of the current user will trigger the callback
registered through `skygear.onUserChanged`, even if the new username/email is
the same as the old one.

</div>

To change the username of the current user:

``` javascript
skygear.saveUser({
  id: skygear.currentUser.id,
  username: 'new-username',
}).then((user) => {
  console.log(user); // updated user object
  console.log('Username is changed to: ', user.username);
}, (error) => {
  console.error(error);
});
```

To change the email of the current user:

``` javascript
skygear.saveUser({
  id: skygear.currentUser.id,
  email: 'new-email@example.com',
}).then((user) => {
  console.log(user); // updated user object
  console.log('Email is changed to: ', user.email);
}, (error) => {
  console.error(error);
});
```

To change the username and email at the same time, you can specify both
`username` and `email` when calling `skygear.saveUser`.

### Changing the password of a user

The currently logged-in user can change his/her own password.
This can be done using the `skygear.changePassword` function.

If the current password is incorrect, the SDK will return an
`InvalidCredentials` error.

``` javascript
skygear.changePassword(currentPassword, newPassword).then((user) => {
  console.log(user); // user object
  console.log('Password has been changed');
}, (error) => {
  console.error(error);
  if (error.error.code === skygearError.InvalidCredentials) {
    // the current password is incorrect
  } else {
    // other kinds of error
  }
});
```

Note: Changing the password of the current user will not trigger the callback
registered through `skygear.onUserChanged`.

#### Invalidating existing access tokens

If you are using the JWT token store, all existing access tokens
will be invalidated when a user changes his/her password.

[Not yet implemented]
If you are not using the JWT token store, you can invalidate existing tokens by
setting `invalidateTokens` to `true`:

``` javascript
skygear.changePassword(currentPassword, newPassword, invalidateTokens=true)
.then((user) => {
  console.log(user); // user object
  console.log('The user has got a new access token');
}, (error) => {
  console.error(error);
});
```

### Password reset

Not yet implemented.

<a name="user-verification"></a>
## User Verification

Not yet implemented.

<a name="social-login"></a>
## Social Login (Facebook, Twitter, etc)

Besides the traditional login method using a password,
Skygear also supports authenticating a user with third party services
such as Facebook and Twitter, using an
[auth provider](/plugin/guide/guide-auth) written with cloud code.

You can use the `skygear.loginWithProvider` method by providing the name of the
auth provider, along with the relevant auth data (an object) necessary for
the authentication, such as the access token obtained from Facebook.

For example, if you have implemented a Facebook auth provider under the name
`com.facebook` using
`@skygear.provides('auth', 'com.facebook')` in the cloud code,
you can log a user in by:

``` javascript
const provider = 'com.facebook';
const authData = {
  'access_token': 'abcdefg',
};
skygear.loginWithProvider(provider, authData);
```

- `provider`: name of the provider, such as `com.facebook` or `com.google`
- `authData`: an object that will be passed as a dictionary to the
  plugin that contains the information necessary to authenticate a user

Note: There is no distinction between signing up and logging in with
an auth provider. the Skygear server will create a new user if the principal ID
(as returned by the auth provider) does not exist in the database.

<a name="user-profile"></a>
## User Profile

Whenever a new user signs up, a user profile is automatically created for
you to track user information other than their username, email or password.

The user profile is created using the record type `user` with
the column `_id` storing the user ID, so you can use it
in the same way as using any other record types.
You can query and update a user's profile by manipulating using
the `User` record type.

Important note: This user profile is created in the public database, i.e.
it is visible to any other user. Therefore you should not store any sensitive
information in this record type. You will need to create another record type
with the private database for information that can only be accessed by the
owner.

### Saving to the current user's profile

``` javascript
// skygear.UserRecord is equivalent to skygear.Record.extend('user')
const modifiedProfile = new skygear.UserRecord({
  '_id': 'user/' + skygear.currentUser.id,
  'language': 'en-US',
  'gender': 'male',
  'age': 20,
});
skygear.publicDB.save(modifiedProfile).then((profile) => {
  console.log(profile); // updated user record
});
```

### Retrieving the current user's profile

``` javascript
const query = new skygear.Query(skygear.UserRecord);
query.equalTo('_id', skygear.currentUser.id);
skygear.publicDB.query(query).then((records) => {
  const profile = records[0];
  console.log(profile);
}, (error) => {
  console.error(error);
});
```

### Retrieving another user's profile by email or username

You can retrieve the public profiles of other users by using their emails or
usernames. You can provide either a single email/username or an array of
emails/usernames.
The promise will be resolved by an array of matched user profiles.

``` javascript
// you can also pass an array of emails
skygear.discoverUserByEmails('ben@skygear.com').then((users) => {
  console.log(users); // array of profiles (user records)
}, (error) => {
  console.error(error);
});
```

``` javascript
// you can also pass an array of usernames
skygear.discoverUserByUsernames('ben').then((users) => {
  console.log(users); // array of profiles (user records)
}, (error) => {
  console.error(error);
});
```
