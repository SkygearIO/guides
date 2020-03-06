# Exchange of Session with User Info at Gateway

## Introduction

Apart from the microservice management functionality, a complete authentication solution **AuthGear** is also shipped with a Skygear subscription. While **AuthGear** takes care of your user management needs, it is also integrated with every microservice deployed on Skygear where user information is automatically provided upon successful login.

To achieve this, you will have to install Skygear JS SDK in your app first. \(\_\_TODO\_\_ add section and link there\)

## How it works under the hood

Say you have a blog posting app, which is formed by a web-based portal and a backend server. Users will have to be authenticated before they can create a new post, or read others'.

To sign up a user for this app, you will have to collect his/her information at your frontend and send it to **AuthGear** with a Skygear SDK function that looks like this:

```typescript
skygear.auth.login(
  <your_email>, 
  <your_password>
).then(() => {
  // some code
});
```

Once the function results in a success, a cookie representing this session \(as the user has signed up successfully, he/she now has a valid authenticated session\) is **automatically** stored in your browser.

Now comes the backend part. New posts can be created by calling an API which requires a user's ID. Since most API clients at the frontend will insert stored cookies along with the request, the user's Skygear session cookie will be provided **automatically** when you call the "create post" API. Notice that the payload carried by the cookie is still a user session, which doesn't fulfil the API's requirement - a user's ID.

Every request fired to a Skygear app will hit the Skygear gateway first. When a user session is detected in the request, the gateway will first ask **AuthGear** if such a user exists, where upon a yes **AuthGear** will provide the user's ID in return. The gateway will then put the user ID into the request header, so that the targeted microservice can get the it.



