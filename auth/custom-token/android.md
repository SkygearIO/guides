---
title: Authenticate with Custom Authentication System
---

[[toc]]

## Introduction

Skygear allows users to sign up or log in with a user-side generated JSON Web Token (JWT). You'll need an authentication server to generate such
JWT, which will be covered below.

By doing so, you can easily integrate Skygear services and resources into
your existing backend services. You can even pass some of the user
information from your server(s) to Skygear.

## Generate custom JWT

To generate a JWT which is valid to Skygear, you'll have to:
1. Configure Skygear's Custom Token Secret on your authentication server
2. Provide claims that are required by Skygear in the JWT

To retrieve custom JWTs from a client application (e.g. your mobile app),
you will have to create a server endpoint on your authentication server.

### Retrieve Custom Token Secret

#### Skygear Cloud

If you are using Skygear Cloud, the Custom Token Secret is generated
automatically. You can retrieve or revoke it on the Developer Portal.

![Custom Token](/assets/common/custom-token.png)

#### Self hosting

If you are hosting Skygear server yourself, the Custom Token Secret has
to be generated and set to the environment variable `CUSTOM_TOKEN_SECRET`
manually.


### Custom JWT claims

The custom JWT must provide all the below claims:

| Claims                                 | Description |
|----------------------------------------|-------------|
| sub (Subject)                          | Authentication server user unique identifier, string between 1-36 characters long (Required) |
| iat (Issued At)                        | The time at which the JWT was issued (Required) |
| exp (Expiration Time)                  | The expiration time on or after which the JWT MUST NOT be accepted for processing (Required) |
| skyprofile (Skygear user profile) | The user profile for signup, will be updated in login |

The claim `skyprofile` is consist of two parts: email and username. With
this you can specify a user's email and username upon the creation of the
custom JWT, allowing you to carry user information from your backend
services to Skygear.

### Example implementations

Here are some example implementations on creation of custom JWTs in
various languages.

#### JavaScript

Install [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken)

```js
var jwt = require('jsonwebtoken');

var userID = "User id of your server";
var customTokenSecret = "CUSTOM_TOKEN_SECRET from portal";
var nowSeconds = Math.floor(Date.now() / 1000);

var token = jwt.sign({
  sub: userID,
  iat: nowSeconds,
  exp: nowSeconds + ( 60 * 60 ), // expire within 1 hour
  skyprofile: {
    email: "user@skygear.io",
    username: "user"
  }
}, customTokenSecret, { algorithm: 'HS256'});
```

#### Python

Install [PyJWT](https://github.com/jpadilla/pyjwt)

```py
import jwt
import time

now_seconds = int(time.time())
user_id = "User id of your server"
custom_token_secret = "CUSTOM_TOKEN_SECRET from portal"

token = jwt.encode({
    'sub': user_id,
    'iat': now_seconds,
    'exp': now_seconds + ( 60 * 60 ), # expire within 1 hour
    'skyprofile': {
        'email': "user@skygear.io",
        'username': "user"
    }
}, custom_token_secret, algorithm='HS256')
```

#### Ruby

Install [ruby-jwt](https://github.com/jwt/ruby-jwt)

```ruby
custom_token_secret = "CUSTOM_TOKEN_SECRET from portal"
user_id = "User id of your server"
now_seconds = Time.now.to_i

payload = {
    :sub => user_id,
    :iat => now_seconds,
    :exp => now_seconds + ( 60 * 60 ), # expire within 1 hour
    :skyprofile => {
      email: => "user@skygear.io",
      username: => "user"
    }
}

JWT.encode payload, custom_token_secret, "HS256"
```

#### Java and Kotlin

Install [jjwt](https://github.com/jwtk/jjwt)

```java
String customTokenSecret = "CUSTOM_TOKEN_SECRET from portal";
String encodedSecret = new String(Base64.encode(customTokenSecret.getBytes(), Base64.DEFAULT));
String userId = "User id of your server";
Date now = new Date();
Date expiration = new Date(now.getTime() + 60 * 60 * 1000);

Claims claims = Jwts.claims()
        .setSubject(userId)
        .setIssuedAt(now)
        .setExpiration(expiration);
Map<String, String> profile = new HashMap<>();
profile.put("email", "user@skygear.io");
profile.put("username", "user");
claims.put("skyprofile", profile);

String token = Jwts.builder()
        .setClaims(claims)
        .signWith(SignatureAlgorithm.HS256, encodedSecret)
        .compact();
```
```kotlin
val customTokenSecret = "CUSTOM_TOKEN_SECRET from portal"
val encodedSecret = String(Base64.encode(customTokenSecret.toByteArray(), Base64.DEFAULT))
val userId = "User id of your server"
val now = Date()
val expiration = Date(now.time + 60 * 60 * 1000)

val claims = Jwts.claims()
  .setSubject(userId)
  .setIssuedAt(now)
  .setExpiration(expiration)
val profile = HashMap<String, String>()
profile.put("email", "user@skygear.io")
profile.put("username", "user")
claims.put("skyprofile", profile)

val token = Jwts.builder()
  .setClaims(claims)
  .signWith(SignatureAlgorithm.HS256, encodedSecret)
  .compact()
```

## Login with custom token

Skygear SDK provides a callback function `loginWithCustomToken` for users
to sign up or log in with a custom JWT.

```java
String customToken = "jwt generated by your authentication server";
skygear.getAuth().loginWithCustomToken(customToken, new AuthResponseHandler() {
    @Override
    public void onAuthSuccess(Record user) {
        Log.i("Skygear loginWithCustomToken", "onAuthSuccess: Got token: " + skygear.getAuth().getCurrentAccessToken());
    }

    @Override
    public void onAuthFail(Error error) {
        Log.e("Skygear loginWithCustomToken", "onAuthFail: Fail with reason: " + error.getMessage());
    }
});
```
```kotlin
val customToken = "jwt generated by your authentication server"
skygear.auth.loginWithCustomToken(customToken, object : AuthResponseHandler() {
    override fun onAuthSuccess(user: Record) {
        Log.i("Skygear", "onAuthSuccess: Got token: ${skygear.auth.currentAccessToken}")
    }

    override fun onAuthFail(error: Error) {
        Log.e("Skygears", "onAuthFail: Fail with reason: ${error.message}")
    }
})
```
