# Sign in with Apple

### Overview

Skygear Auth supports Sign in with Apple \(SIWA\).

To configure SIWA for Skygear Auth, you have to fulfill the following prerequisites:

1. Register an Apple Developer Account. Apple Enterprise Account does not support SIWA.
2. Register your own domain.
3. Your domain must be able to send and receive emails.
4. Set up [SPF](https://en.wikipedia.org/wiki/Sender_Policy_Framework) for your domain.
5. Set up [DKIM](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail) for your domain.
6. On Apple Developer, follow the instructions there and verify your domain.
7. Create an App ID with SIWA enabled.
8. [Create a Services ID with SIWA enabled](sign-in-with-apple.md#create-a-services-id-with-siwa-enabled).
9. Create a Key with SIWA enabled. Keep the private key safe. You need to provide this later.

The next step is to choose a custom URI scheme for your frontend apps. This URI scheme must not collide with other applications in this earth. So it is better to choose a longer one.

To configure your iOS app, perform the following tasks:

1. Ensure the bundle ID of your iOS app is the App ID above.
2. Add the capability "Sign in with Apple" to your iOS app.
3. [Declare your custom URI scheme in Info.plist.](sign-in-with-apple.md#declare-your-custom-uri-scheme-in-info-plist)
4. [Forward openURL callback in AppDelegate.](sign-in-with-apple.md#forward-openurl-callback-in-appdelegate)

To configure your Android app, perform the following tasks:

1. [Declare an callback activity in AndroidManifest.xml.](sign-in-with-apple.md#declare-an-callback-activity-in-androidmanifest-xml)

The final task is to [configure Skygear Auth](sign-in-with-apple.md#configure-skygear-auth).

Now you can write [the code to trigger SIWA](sign-in-with-apple.md#sample-code).

### Create a Services ID with SIWA enabled

The Return URLs can have different domain than the domain used for sending and receiving emails.

To test locally on your own machine, you can edit `/etc/hosts` to force a domain to be resolved to `127.0.0.1`.

For example

```
# Add the following line to /etc/hosts so that myapp.com resolves to 127.0.0.1
127.0.0.1 myapp.com
```

And then you can add the Return URL. Depending on whether you are using Auth UI or Auth API, the URL is different.

```
# If you are using Auth UI, add this one.
https://myapp.com/sso/oauth2/callback/<provider-id>
# Otherwise if you are using Auth API, add this one.
https://myapp.com/_auth/sso/<provider-id>/auth_handler
```

`provider-id` is just an ID you provide in the configuration to allow Skygear Auth to identify a provider.

### Declare your custom URI scheme in Info.plist

```markup
<!-- Declare that the application can respond to the callback URL. -->
<key>CFBundleURLTypes</key>
<array>
       <dict>
               <key>CFBundleTypeRole</key>
               <string>Editor</string>
               <key>CFBundleURLSchemes</key>
               <array>
                       <string>myapp</string>
               </array>
       </dict>
</array>
```

### Forward openURL callback in AppDelegate

```objectivec
#import "SGSkygearReactNative.h"

- (BOOL)application:(UIApplication *)application
   openURL:(NSURL *)url
   options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
{
  return [SGSkygearReactNative application:application openURL:url options:options];
}
```

### Declare an callback activity in AndroidManifest.xml

```markup
<!-- Declare that the application can respond to the callback URL. -->
<!-- The activity is implemented by the SDK. -->
<activity
  android:name="io.skygear.reactnative.OAuthRedirectActivity">
  <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data
      android:scheme="myapp"
      android:host="host" />
  </intent-filter>
</activity>
```

### Configure Skygear Auth

```yaml
sso:
  oauth:
    allowed_callback_urls:
    # The callback URL must match the ones declared in the iOS application and in the Android application.
    - "myapp://host/path"
    # You need to configure two providers. One for iOS app and one for other platforms.
    providers:
      # id determines how your client code reference the provider.
    - id: apple-app
      # type is apple obviously.
      type: apple
      # This provider configuration is for iOS application.
      # So fill in your App ID here.
      client_id: 'your-sign-in-with-apple-enabled-app-id'
      # Fill in the Key as client_secret
      client_secret: |
        -----BEGIN PRIVATE KEY-----
        The contents of the Key here.
        -----END PRIVATE KEY-----
      # Fill in your Apple Developer Account team ID.
      team_id: 'your-apple-developer-account-team-id'
      # Fill in the ID of the Key.
      key_id: 'the-id-of-the-key'
    - id: apple-web
      type: apple
      # This provider configuration is for other platforms.
      # So fill in your Services ID here.
      client_id: 'your-sign-in-with-apple-enabled-services-id'
      # The remaining fields are the same as the above provider.
      client_secret: |
        -----BEGIN PRIVATE KEY-----
        The contents of the Key here.
        -----END PRIVATE KEY-----
      team_id: 'your-apple-developer-account-team-id'
      key_id: 'the-id-of-the-key'

```

### Sample Code

```typescript
import { getSystemVersion } from "react-native-device-info";
// This button must follow Apple design guidelines.
// This library provides a conforming button.
// https://github.com/expo/expo/blob/master/packages/expo-apple-authentication/src/AppleAuthenticationButton.tsx
import SignInWithAppleButton from "./SignInWithAppleButton";

function getMajorSystemVersion(): number {
  const version = getSystemVersion();
  const parts = version.split(".");
  return parseInt(parts[0], 10);
}

function LoginScreen() {
  // For a React Native app, Sign in with Apple must provide two experiences.
  //
  // On iOS >= 13, the developer must use loginApple which uses AuthenticationServices
  // to provide a native experience.
  //
  // On other iOS versions and Android, the developer must use loginOAuthProvider which is the standard
  // web-based OpenID Connect flow.
  const onPress = useCallback(() => {
    if (Platform.OS === "ios" && getMajorSystemVersion() >= 13) {
      skygear.auth.loginApple("apple-app", "myapp://host/path");
    } else if (Platform.OS === "ios" || Platform.OS === "android") {
      skygear.auth.loginOAuthProvider("apple-web", "myapp://host/path");
    }
  }, []);
  return (
    <View>
      <SignInWithAppleButton onPress={onPress} />
    </View>
  );
}
```

