---
title: Social Login
description: User login with 3rd party authentication in iOS.
---

[[toc]]

Skygear allows you to authenticate users using their 3rd party accounts.

## Setup

Currently following providers are supported. Please find the setup instructions below:

- [Google][google]
- [Facebook][facebook]
- [LinkedIn][linkedin]
- [Instagram][instagram]

### Login with 3rd party account

Skygear SDK supports two kinds of login flow:

- [Login with web flow][login-with-web-flow]
- [Login with access token][login-with-access-token]

## Login with web flow

1. Configure Callback URLs

    The callback URL is URL of your application that Skygear redirects to after user is authenticated. You can configure the app scheme by adding following lines into the `Info.plist` and replace `${YOUR_APP_SCHEME}` with your URL scheme.

    ```xml
    <key>CFBundleURLTypes</key>
    <array>
        <dict>
            <key>CFBundleURLName</key>
            <string>skygear</string>
            <key>CFBundleURLSchemes</key>
            <array>
                <string>${YOUR_APP_SCHEME}</string>
            </array>
        </dict>
    </array>
    ```

    Skygear SDK will redirect back to your application after user is authenticated through the following link. For production app, please make sure your Skygear portal *Allow redirect URLs* contains the below link. All URLs will be allowed, if *Allow redirect URLs* is empty.

    ```
    ${YOUR_APP_SCHEME}://skygeario.com/auth_handler
    ```

    ![Allow redirect URLs](/assets/common/sso-allow-redirect-urls.png)

2. Trigger login with web flow

    By calling `loginOAuthProvider`, Skygear SDK will complete the login flow by using browser to login user.

    ```obj-c
    NSString *providerID = @"google"; // the provider name in lowercase

    NSDictionary *options = @{
        // scheme: you app scheme
        @"scheme": @"${YOUR_APP_SCHEME}",
        // scope (optional): overwrite the portal default permissions if it is provided
        @"scope": @[@"email"]
    };

    SKYContainer *container = [SKYContainer defaultContainer];
    [container.auth loginOAuthProvider:providerID
                               options:options
                     completionHandler:^(SKYRecord *user, NSError *error) {
                        if (error) {
                            NSLog(@"error login with oauth provider: %@", error);
                            return;
                        }

                        NSLog(@"sign up successful");
                        NSLog(@"user record: %@", user);
                    }];
    ```

    ```swift
    let providerID = "google"; // the provider name in lowercase

    let options = [
        // scheme: you app scheme
        "scheme": "skygearexample",
        // scope (optional): will overwrite portal default permissions if it is provided
        "scope": ["email"]
    ] as [String: Any]

    SKYContainer.default().auth.loginOAuthProvider(providerID, options: options) {(user, error) in
        if error != nil {
            print("error login with provider: \(error)")
            return
        }

        print("Sign up successfully")
        print("Login user \(user)")
        print("email: \(user!["email"])")
    }
    ```

## Login with access token

If you obtain the access token from 3rd party provider SDK, you can login to provider as follows:

```obj-c
NSString *providerID = @"google"; // the provider name in lowercase
NSString *accessToken = @"access token from 3rd party sdk";

SKYContainer *container = [SKYContainer defaultContainer];
[container.auth loginOAuthProvider:providerID
                       accessToken:accessToken
                 completionHandler:^(SKYRecord *user, NSError *error) {
                    if (error) {
                        NSLog(@"error login with provider: %@", error);
                        return;
                    }

                    NSLog(@"sign up successful");
                    NSLog(@"user record: %@", user);
                }];
```

```swift
let providerID = "google"; // the provider name in lowercase
let accessToken = "access token from 3rd party sdk"

SKYContainer.default().auth.loginOAuthProvider(providerID, accessToken: accessToken) {(user, error) in
    if error != nil {
        print("error login with provider: \(error)")
        return
    }

    print("Login user %@", user.debugDescription)
}
```

## Link current user with web login flow

Follow steps in [Login with web flow][login-with-web-flow] and call `linkOAuthProvider` instead of `loginOAuthProvider` after user login.

```obj-c
NSString *providerID = @"google"; // the provider name in lowercase

NSDictionary *options = @{
    // scheme: you app scheme
    @"scheme": @"${YOUR_APP_SCHEME}",
    // scope (optional): will overwrite portal default permissions if it is provided
    @"scope": @[@"email"]
};

SKYContainer *container = [SKYContainer defaultContainer];
[container.auth linkOAuthProvider:providerID
                          options:options
                completionHandler:^(SKYRecord *user, NSError *error) {
                    if (error) {
                        NSLog(@"error link with provider: %@", error);
                        return;
                    }

                    NSLog(@"sign up successful");
                    NSLog(@"user record: %@", user);
                }];
```

```swift
let providerID = "google"; // the provider name in lowercase

let options = [
    // scheme: you app scheme
    "scheme": "skygearexample",
    // scope (optional): will overwrite portal default permissions if it is provided
    "scope": ["email"]
] as [String: Any]

SKYContainer.default().auth.linkOAuthProvider(providerID, options: options) {(error) in
    if error != nil {
        print("error link with provider: \(error)")
        return
    }

    print("link user with google account successfully")
}
```

## Link current user with access token

If you obtain the access token from 3rd party provider SDK, you can link the current user to provider as follows:

```obj-c
NSString *providerID = @"google"; // the provider name in lowercase
NSString *accessToken = @"access token from 3rd party sdk";

SKYContainer *container = [SKYContainer defaultContainer];
[container.auth linkOAuthProvider:providerID
                       accessToken:accessToken
                 completionHandler:^(NSError *error) {
                    if (error) {
                        NSLog(@"error link with provider: %@", error);
                        return;
                    }

                    NSLog(@"link user with google account successfully");
                }];
```

```swift
let providerID = "google"; // the provider name in lowercase
let accessToken = "access token from 3rd party sdk"

SKYContainer.default().auth.linkOAuthProvider(providerID, accessToken: accessToken) {(error) in
    if error != nil {
        print("error link with provider: \(error)")
        return
    }

    print("link user with google account successfully")
}
```

## Unlink current user with provider

To unlink current user with the provider.

```obj-c
NSString *providerID = @"google"; // the provider name in lowercase
NSString *accessToken = @"access token from 3rd party sdk";

SKYContainer *container = [SKYContainer defaultContainer];
[container.auth linkOAuthProvider:providerID
                 completionHandler:^(NSError *error) {
                    if (error) {
                        NSLog(@"error unlink with provider: %@", error);
                        return;
                    }

                    NSLog(@"unlink user google account successfully");
                }];
```

```swift
let providerID = "google"; // the provider name in lowercase

SKYContainer.default().auth.unlinkOAuthProvider(providerID) {(error) in
    if error != nil {
        print("error unlink with provider: \(error)")
        return
    }

    print("unlink user google account successfully")
}
```

## Get the 3rd party provider user profiles

`getOAuthProviderProfilesWithCompletionHandler` will return a dictionary of user connected provider profiles. The keys are the providers ID and the values are the profile dictionary. Every time when user login, Skygear will request and update the 3rd party user profile. This is useful to identify if user links with the provider.


```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
[container.auth getOAuthProviderProfilesWithCompletionHandler:^(NSDictionary *result, NSError *error) {
    if (error) {
        NSLog(@"error to get provider profile: %@", error);
        return;
    }

    NSString *providerID = @"google"; // the provider name in lowercase
    if let profile = result?[providerID] {
        print("Current user linked with google")
        print("Google profile \(profile)")
    } else {
        print("Google profile \(profile)")
    }
}];
```


```swift
SKYContainer.default().auth.getOAuthProviderProfiles {(result, error) in
    if error != nil {
        print("error to get provider profile: \(error)")
        return
    }

    if let profile = result?["google"] {
        print("Google profile \(profile)")
        return
    }
}
```

[login-with-web-flow]: #login-with-web-flow
[login-with-access-token]: #login-with-access-token
[facebook]: /guides/auth/social-login/facebook/
[google]: /guides/auth/social-login/google/
[linkedin]: /guides/auth/social-login/linkedin/
[instagram]: /guides/auth/social-login/instagram/
