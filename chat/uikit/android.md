---
title: Skygear Chat Android UIKit
---

[[toc]]

## About UIKit
UIKit provides the `ConversationActivity`, the main UI of a conversation.
The view of a `ConversationActivity` is `ConversationView`. The view consists of the following components.

1. Title Bar
2. Avatar (Outgoing)
3. Message (Outgoing)
4. Avatar (Incoming)
5. Message (Incoming)
6. Camera Button
7. Text Input
8. Voice Button

![ConversationView Overview](/assets/chat/uikit/android/conversation-view-overview.png)



## Prerequisite
### Step 1: Add Skygear and Skygear Chat to your project

Please follow the steps at [Quick Start](https://docs.skygear.io/guides/chat/quick-start/android/).

### Step 2: Add Skygear Chat UIKit to your project


1. add `io.skygear.plugings:chat_ui` in `build.gradle`.

```gradle
dependencies {
    // other dependencies
    compile 'io.skygear.plugins:chat_ui:+'
}
```

2. You will be hinted for a project sync as you have updated the gradle files. The Skygear Android SDK will have been installed when the sync is completed.

3.  Import UIKit in your source code as follows

```java
import io.skygear.plugins.chat.ui.*;
```

```kotlin
import io.skygear.plugins.chat.ui.*
```


## Creating a conversation UI
The following code block creates a `ConversationActivity` of a conversation `c`.

```java
Intent intent = new Intent(this, ConversationActivity.class);
// c is an instance of an io.skygear.plugins.chat.Conversation
intent.putExtra(ConversationActivity.ConversationIntentKey, c.toJson().toString());
startActivity(intent);
```

```kotlin
val intent = Intent(this, ConversationActivity::class.java)
// c is an instance of an io.skygear.plugins.chat.Conversation
intent.putExtra(ConversationActivity.ConversationIntentKey, c.toJson().toString())
startActivity(intent)
```

## Customizations
`ConversationActivity` can be customized via

1. specifying XML resource of a `ConversationView`,
2. customizing attribute in the XML resource and
3. implementing interface and adapters.


## Attribute Customization

### Specifying XML resource

Suppose you have created the following XML resource, `custom_view`, under `/res/layout` directory.

```xml
<io.skygear.plugins.chat.ui.ConversationView
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    uikit:avatarBackgroundColor="@color/common_google_signin_btn_text_light"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:uikit="http://schemas.android.com/apk/res-auto"/>
```

Then you need to specify the corresponding resource ID in intent.

```java
Intent intent = new Intent(this, ConversationActivity.class);
intent.putExtra(ConversationActivity.ConversationIntentKey, c.toJson().toString());
intent.putExtra(ConversationActivity.LayoutIntentKey, R.layout.custom_view);
startActivity(intent);
```

```kotlin
val intent = Intent(this, ConversationActivity::class.java)
intent.putExtra(ConversationActivity.ConversationIntentKey, c.toJson().toString())
intent.putExtra(ConversationActivity.LayoutIntentKey, R.layout.custom_view)
startActivity(intent)
```

The following XML attributes are supported.

#### Avatar
|Attribute|Description|Type|
|---------|-----------|------|
|`avatarBackgroundColor`|Background color of the avatar|Color|
|`avatarTextColor`|Text color of the avatar|Color|
|`userAvatarType`|Avatar Type, either showing initial of the username field or image|String, "initial" or "image"|
|`userNameField`|Field name of the username in `user` table|String, default "username"|
|`userAvatarField`|Field name of the avatar image in `user` table|String|
|`avatarHiddenForIncomingMessages`|Hide incoming message avatar if true|Boolean, default False|
|`avatarHiddenForOutgoingMessages`|Hide outgoing message avatar if true|Boolean, default True|

#### Message
|Attribute|Description|Type|
|---------|-----------|------|
|`messageSenderTextColor`|Text Color of the message Sender|Color|
|`backgroundColorForIncomingMessages`|Incoming messages background color|Color|
|`backgroundColorForOutgoingMessages`|Outgoing messages background color|Color|
|`textColorForIncomingMessages`|Incoming messages text color|Color|
|`textColorForOutgoingMessages`|Outgoing messages text color|Color|
|`voiceMessageButtonColorForOutgoingMessages`|Outgoing voice message button color|Color|
|`voiceMessageButtonColorForIncomingMessages`|Incoming voice message button color|Color|

#### Status Text & Date Time Format
|Attribute|Description|Type|
|---------|-----------|------|
|`delivering`|Delivering text|String, default "Delivering"|
|`delivered`|Delivered text|String, default "Delivered"|
|`someRead`|Some read text|String, default "Some Read"|
|`allRead`|All read text|String, default "All Read"|
|`failed`|Failed text|String, default "Failed"|
|`hint`|Place holder of the input text|String, default "Type a message..."|
|`statusTextColorForIncomingMessages`|Status text color (Incoming)|Color|
|`statusTextColorForOutgoingMessages`|Status text color (Outgoing)|Color|
|`timeTextColorForOutgoingMessages`|Time text color (Incoming)|Color|
|`timeTextColorForIncomingMessages`|Time text color (Outgoing)|Color|


#### Feature On/Off
|Attribute|Description|Type|
|---------|-----------|------|
|`cameraButtonShouldShow`|Show camera button if true|Boolean, default: True|
|`voiceMessageButtonShouldShow`|Show voice message button if true|Boolean, default: True|
|`messageStatusShouldShow`|Show message status if true|Boolean, default: True|

### Title Option
UIKit provides two title options

1. `DEFAULT` and
2. `OTHER_PARTICIPANTS`

If `DEFAULT` is set, then the title bar displays `title` field value of the conversation record. If  `OTHER_PARTICIPANTS` is set,
then the title bar displays participants' usernames (excluding the current user) in the title bar.

```java
Intent intent = new Intent(this, ConversationActivity.class);
intent.putExtra(ConversationActivity.ConversationIntentKey, c.toJson().toString());
intent.putExtra(ConversationActivity.TitleOptionIntentKey, ConversationTitleOption.OTHER_PARTICIPANTS);
startActivity(intent);
```

```kotlin
val intent = Intent(this, ConversationActivity::class.java)
intent.putExtra(ConversationActivity.ConversationIntentKey, c.toJson().toString())
intent.putExtra(ConversationActivity.TitleOptionIntentKey, ConversationTitleOption.OTHER_PARTICIPANTS)
startActivity(intent)
```

## Adapter
UIKit provides `ConversationViewAdapter` to enable remote HTTP background image in `ConversationActivity`.

```java
ConversationViewAdapter adapter = new ConversationViewAdapter() {
    @Override
    public String backgroundImageURL(Conversation conversation) {
        return "https://skygear.io/static/images/skygear-logo.png";
    }
};
```

```kotlin
val adapter = object : ConversationViewAdapter() {
    override fun backgroundImageURL(conversation: Conversation): String {
        return "https://skygear.io/static/images/skygear-logo.png"
    }
}
```

We can set the adapter via intent.

```java
Intent intent = new Intent(this, ConversationActivity.class);
intent.putExtra(ConversationActivity.ConversationIntentKey, c.toJson().toString());
intent.putExtra(ConversationActivity.ConversationViewAdapterIntentKey, adapter);
startActivity(intent);
```

```kotlin
val intent = Intent(this, ConversationActivity::class.java)
intent.putExtra(ConversationActivity.ConversationIntentKey, c.toJson().toString())
intent.putExtra(ConversationActivity.ConversationViewAdapterIntentKey, adapter)
startActivity(intent)
```


## Avatar Adapter
Alternatively, an avatar can also be customized via defining a `AvatarAdapter`.

The following is a customized avatar adapter.

```java
AvatarAdapter adapter = new AvatarAdapter() {
    @Override
    public createAvatarView(LayoutInflater inflater, ViewGroup viewGroup) {
        TextView textView = new TextView(viewGroup.getContext());
        textView.setLayoutParams(new ViewGroup.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT));
        return textView;
    }

    @Override
    public void bind(View view, Conversation conversation, Message message, User user) {
        TextView tv = (TextView) view;
        tv.setBackgroundColor(Color.RED);
        tv.setText(conversation.title.substring(0, 2) + user.getName());
    }
};
```

```kotlin
val adapter = object : AvatarAdapter() {
    override fun createAvatarView(inflater: LayoutInflater, viewGroup: ViewGroup): View? {
        val textView = TextView(viewGroup.context)
        textView.layoutParams = ViewGroup.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT)
        return textView
    }

    override fun bind(view: View, conversation: Conversation?, message: io.skygear.plugins.chat.ui.model.Message, user: User?) {
        val tv = view as TextView
        tv.setBackgroundColor(Color.RED)
        tv.text = conversation.title?.substring(0, 2) + user.getName()
    }
}
```

We can set the adapter via intent.

```java
Intent intent = new Intent(this, ConversationActivity.class);
intent.putExtra(ConversationActivity.ConversationIntentKey, c.toJson().toString());
intent.putExtra(ConversationActivity.AvatarAdapterIntentKey, adapter);
startActivity(intent);
```

```kotlin
val intent = Intent(this, ConversationActivity::class.java)
intent.putExtra(ConversationActivity.ConversationIntentKey, c.toJson().toString())
intent.putExtra(ConversationActivity.AvatarAdapterIntentKey, adapter)
startActivity(intent)
```


