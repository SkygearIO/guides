---
title: Quick Start
descripition: Adding Skygear to your Android apps
---

[[toc]]

This guide will walk you through the steps to add Skygear to your Android project from scratch.

:::note
Alternatively, you might want to get started with Skygear using:
* [Skygear Scaffolding Template for Android](https://github.com/SkygearIO/skygear-Scaffolding-Android)
* [Android API Samples Project](https://github.com/SkygearIO/skygear-SDK-Android/tree/master/skygear_example)
* [List of Android demo / tutorials at github.com/skygear-demo](https://github.com/search?q=topic%3Askygear-android+org%3Askygear-demo)
:::

## Prerequisite

- [Android Studio](https://developer.android.com/studio/index.html)
- A Skygear account. Sign up [here](https://portal.skygear.io/signup).

## Step 1: Install the Skygear Android SDK

1. Create a new Android Studio project or use your existing one.
2. In `build.gradle` of the **project**, include the JCenter Maven repository.

```java
allprojects {
   repositories {
       jcenter()
   }
}
```

3. Then, in `build.gradle` of the **module**, add Skygear as a dependency by adding `io.skygear:skygear+` in the dependency list.

```java   
dependencies {
    // other dependencies
    compile 'io.skygear:skygear:+'
}
```

4. You will be hinted for a project sync as you have updated the `gradle` files. The Skygear Android SDK will have been installed when the sync is completed.

![Skygear Android SDK](/assets/android/android-project-sync.png)

## Step 2: Configure Skygear in your app

### Method 1: Create an `Application` that extends `SkygearApplication`

If your existing project does not have any extension, you can configure Skygear by creating an `Application ` that extends `SkygearApplication`.

Assume your custom application is defined in `MyApplication.java`, add the following lines to the file.

```java
import io.skygear.skygear.SkygearApplication;

public class MyApplication extends SkygearApplication {
    @Override
    public String getSkygearEndpoint() {
        return "https://<your-app-name>.skygeario.com/";
    }

    @Override
    public String getApiKey() {
        return "<your-api-key>";
    }
}
```
:::note
You can get your server endpoints and the API keys in the _info page_ in your [developer portal](https://portal.skygear.io/apps) after signing up for the [Skygear Cloud Services](https://portal.skygear.io/signup).
:::

### Method 2: Configure Skygear when the app starts

If you already have your own custom applications, you can configure Skygear in the `onCreate` function. Your `MyApplication.java` should look like this:

```java
import io.skygear.skygear.Container;
import io.skygear.skygear.Configuration;

public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        Configuration config = new Configuration.Builder()
                .endPoint("https://<your-app-name>.skygeario.com/")
                .apiKey("<your-api-key>")
                .build();

        Container.defaultContainer(this).configure(config);
    }
}

```

:::note
You can get your server endpoints and the API keys in the _info page_ in your [developer portal](https://portal.skygear.io/apps) after signing up for the [Skygear Cloud Services](https://portal.skygear.io/signup).
:::

## Step 3: Update the app manifest
For the Skygear Android SDK to work, make sure the following two items are set in `AndroidManifest.xml`.

1. Include the `android.permission.INTERNET` permission. (So that the Skygear Android SDK can communicate with the Skygear server)
2. Specify the main Activity Class (an activity subclass) as the `android:name`  attribute at the activity tag.

```java
// Your AndroidManifast.xml should have the followings lines

<uses-permission android:name="android.permission.INTERNET" />
<application
    android:name=".MyApplication">
</application>
```


## Step 4: Create your first record in Skygear
Now, let's create a record in the Skygear database to see if the SDK has been installed successfully.

Add the following lines in the `onCreate` function and run your app in a simulator. The record will be created when you run your app.

:::tips
Practically the codes should not be structured this way. It is for demo only.
:::

```java
// import the modules
import io.skygear.skygear.AuthResponseHandler;
import io.skygear.skygear.Error;
import io.skygear.skygear.Record;
import io.skygear.skygear.RecordSaveResponseHandler;
import io.skygear.skygear.SkygearApplication;
import io.skygear.skygear.Container;
import io.skygear.skygear.User;

// Every record in Skygear must be owned by a user
// For testing purpose, we have used signupAnonmously to create a record
// Visit the user authetication documentation to learn more
// https://docs.skygear.io/guides/auth/basics/android/
@Override
public void onCreate() {
    super.onCreate();
    final Container skygear = Container.defaultContainer(this);
    skygear.signupAnonymously(new AuthResponseHandler() {
        @Override
        public void onAuthSuccess(User user) {
            Log.i("MyApplication", "Signup successfully");

            // Create Record Type "test" and put "Hello world" as value of key "content"
            // Advanced: Skygear Server will create a table "test" and appropriate
            //           columns in PostgreSQL in Development mode.
            Record test = new Record("test");
            test.set("content", "Hello world");

            skygear.getPublicDatabase().save(test, new RecordSaveResponseHandler() {
                @Override
                public void onSaveSuccess(Record[] records) {
                    Log.i("MyApplication", "Record saved");
                }

                @Override
                public void onPartiallySaveSuccess(Map<String, Record> successRecords, Map<String, Error> errors) {
                    Log.i("MyApplication", "Some records are failed to save");   
                }


                @Override
                public void onSaveFail(Error error) {
                    Log.w("MyApplication", "Failed to save: " + error.getMessage(), error);
                }
            });
        }

        @Override
        public void onAuthFail(Error error) {
            Log.w("MyApplication", "Failed to signup: " + error.getMessage(), error);
        }
    });
}
```

If the record is created successfully, you should see the record "Hello World" in your database table "test".

You can access your database using the data browser we provide. It can be found in the _info_ page in your [developer portal](https://portal.skygear.io/apps).

![Skygear portal](/assets/common/open-database-in-web-browser.png)

This is how your data browser will look like.

![Web database viewer](/assets/common/quickstart-database-viewer.png)

:::tips
You can access Skygear database in 3 ways.
1. Web data browser: It can be found in from  _info_ page in your [developer portal](https://portal.skygear.io/apps).
2. PostgreSQL client: Skygear database can viewed in any PostgreSQL client. Get the connection string from the _info_ page in your [developer portal](https://portal.skygear.io/apps). We recommend using [Postico](https://eggerapps.at/postico/).
3. Skygear CMS: Skygear CMS is a business-user friendly web interface that allows users to edit the data in the database. To use the CMS, you have to enable it in the _plug-ins_ page in the [developer portal](https://portal.skygear.io/apps). Your CMS URL is https://insert-your-app-name.skygeario.com/cms.

:::

Hurray! Everything should be in the right place from here.

## What's next?
Next, you may want to learn more about:
* [Skygear Cloud Database basics](https://docs.skygear.io/guides/cloud-db/basics/android/)
* [Skygear User Auth Introduction](https://docs.skygear.io/guides/auth/basics/android/)
* [Skygear Chat Introduction](https://docs.skygear.io/guides/chat/basics/android/)
* [Other Skygear Guides](https://docs.skygear.io/)
