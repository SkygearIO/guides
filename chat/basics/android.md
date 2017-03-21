---
title: Skygear Chat
---

# Skygear Chat Basic (Android)
Skygear Chat is a collection of APIs to help you build Chat apps much easier.

## Enabling on Chat Plugin on Skygear Portal

To start using Skygear Chat feature, make sure you have already enabled **Chat** in the **Plug-ins** tab in your [Skygear Portal](https://portal.skygear.io).

![](https://i.imgur.com/g2TOPHu.png)

To set up a Android project using Skygear, please refer to the [Quick Start](https://docs.skygear.io/guides/get-started/android/).

You also need to install the Chat plugin SDK into your project via Gradle.

Add `io.skygear.plugins:chat:+` dependency in your `build.gradle` **of the module**.

```Java
dependencies {
    // other dependencies
    compile 'io.skygear.plugins:chat:+'
}
```

You will be hinted for a project sync as you have updated the `gradle` files. The Skygear Android SDK will have been installed when the sync is completed.

You will interact with the Chat Container to access features of the Chat API. Here is how you can get the `ChatContainer` instance:

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);
```

## Creating conversations
In order to send messages, you need to create a conversation first. You can consider **conversations** as chatrooms or channels in your application.

There are two types of conversations in Skygear:
- **Direct Conversation**, support chatting between one user to another
- **Group Conversation**, support chatting among 2 or more users

### Creating direct conversations

You can use [`createDirectConversation`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#createDirectConversation(java.lang.String,%20java.lang.String,%20java.util.Map,%20io.skygear.plugins.chat.SaveCallback)) to create a conversation with another user. Please specify the user ID as `userID`.

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.createDirectConversation(userBen.getId(), "Chat with Ben", null, new SaveCallback<Conversation>() {
    @Override
    public void onSucc(@Nullable Conversation conversation) {
        Log.i("MyApplication", "Created: " + conversation.getId());
    }

    @Override
    public void onFail(@Nullable String failReason) {
        Log.w("MyApplication", "Failed to save: " + failReason);
    }
});
```

This example shows how to create a direct chat between the current user and `userBen`.

You can also set up optional `metadata` in your conversation for custom configurations.

### Creating group conversations
Besides direct chats, you can also create a group conversation with 2 or more people.

Instead of a single `userID` as the parameter, `createConversation` accepts a list of `userIDs` in a Set of String . A new conversation will be created with the IDs given as participants.

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

HashSet<String> users = new HashSet<String>();
users.add(skygear.getCurrentUser().getId());
users.add(userBen.getId());
users.add(userChris.getId());

chatContainer.createConversation(users, "Random Conversation", null, null, new SaveCallback<Conversation>() {
    @Override
    public void onSucc(@Nullable Conversation conversation) {
        Log.i("MyApplication", "Created: " + conversation.getId());

    }

    @Override
    public void onFail(@Nullable String failReason) {
        Log.w("MyApplication", "Failed to save: " + failReason);
    }
});
```

### Creating group chat by distinct participants *(distinctByParticipants)*

By default, if you try to create conversations with same list of participants with `createConversation`. You will eventually create different conversations with the identical participants.

This may or may not be a desire behavior in your application depends on your app design.


In the options, you can pass in `DISTINCT_BY_PARTICIPANTS` to set the conversations to be distinct by participants.

By default, `DISTINCT_BY_PARTICIPANTS` is set to be `false`.

You can also set it with a separated call on  [`setConversationDistinctByParticipants`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#setConversationDistinctByParticipants(io.skygear.plugins.chat.Conversation,%20boolean,%20io.skygear.plugins.chat.SaveCallback)).

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.setConversationDistinctByParticipants(conversation,
    true,
    new SaveCallback<Conversation>() {
    @Override
    public void onSucc(@Nullable Conversation conversation) {
        Log.i("MyApplication", "setConversationDistinctByParticipants: " + conversation.getId());
    }

    @Override
    public void onFail(@Nullable String failReason) {
        Log.w("MyApplication", "Failed to setConversationDistinctByParticipants: " + failReason);
    }
});
```

Since we have set `distinctByParticipants` to be true, upon calling the above function twice, you will receive an error as duplicated conversations.

### Setting admins

In options you can pass in `ADMIN_IDS` to set the admin list.


You can add an user as admin via using [`addConversationAdmin`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#addConversationAdmin(io.skygear.plugins.chat.Conversation,%20java.lang.String,%20io.skygear.plugins.chat.SaveCallback)) and [`setConversationAdminIds`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#setConversationAdminIds(io.skygear.plugins.chat.Conversation,%20java.util.Set,%20io.skygear.plugins.chat.SaveCallback))


```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.addConversationAdmin(conversation,
    skygear.getCurrentUser().getId(),
    new SaveCallback<Conversation>() {
            @Override
            public void onSucc(@Nullable Conversation conversation) {
                Log.i("MyApplication", "Admin set: " + conversation.getId());
            }

            @Override
            public void onFail(@Nullable String failReason) {
                Log.w("MyApplication", "Failed to save: " + failReason);
            }
        });
```

In the above conversation, the current user will be the admin of the conversation.

Example of `setConversationAdminIds`:

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

HashSet<String> admins = new HashSet<String>();
admins.add(skygear.getCurrentUser().getId());
admin.add(userBen.getId());


chatContainer.setConversationAdminIds(conversation,
    admins,
    new SaveCallback<Conversation>() {
        @Override
        public void onSucc(@Nullable Conversation conversation) {
            Log.i("MyApplication", "Admin set: " + conversation.getId());
        }

        @Override
        public void onFail(@Nullable String failReason) {
            Log.w("MyApplication", "Failed to save: " + failReason);
        }
});
```

In the above example, `userBen` and the current will be set as the admin of the conversation.

### Removing Admins
To remove admins from a conversation, you can call [`removeConversationAdmin`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#removeConversationAdmin(io.skygear.plugins.chat.Conversation,%20java.lang.String,%20io.skygear.plugins.chat.SaveCallback)).

### Fetching existing conversations

There are `UserConversation` and `Conversation` in Skygear Chat.

In normal situation, we query `UserConversation` for display and create a `UserConversation` to a start conversation. User-specific information such as unread count, is available in `UserConversation`.

`Conversation` contains information of a conversation that is shared among all participants, such as `participantIds` and `adminIds`.

There are five methods for fetching the existing conversations:

- [`getUserConversation(ChatUser user, java.lang.Boolean getLastMessages, GetCallback<java.util.List<UserConversation>> callback)`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#getUserConversation(io.skygear.plugins.chat.ChatUser,%20java.lang.Boolean,%20io.skygear.plugins.chat.GetCallback))

- [`getUserConversation(ChatUser user, GetCallback<java.util.List<UserConversation>> callback)`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#getUserConversation(io.skygear.plugins.chat.ChatUser,%20io.skygear.plugins.chat.GetCallback))

- [`getUserConversation(Conversation conversation, ChatUser user, GetCallback<UserConversation> callback)`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#getUserConversation(io.skygear.plugins.chat.Conversation,%20io.skygear.plugins.chat.ChatUser,%20io.skygear.plugins.chat.GetCallback))

- [`getUserConversation(Conversation conversation, GetCallback<UserConversation> callback)`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#getUserConversation(io.skygear.plugins.chat.Conversation,%20io.skygear.plugins.chat.GetCallback))

- [`	getUserConversation(GetCallback<java.util.List<UserConversation>> callback)`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#getUserConversation(io.skygear.plugins.chat.GetCallback))

Fetch the all conversations which involves the current user. You need to set the parameter `getLastMessages` to be `true` in [`getUserConversation(ChatUser user, java.lang.Boolean getLastMessages, GetCallback<java.util.List<UserConversation>> callback)`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#getUserConversation(io.skygear.plugins.chat.ChatUser,%20java.lang.Boolean,%20io.skygear.plugins.chat.GetCallback))


```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.getUserConversation(chatUser,
    true ,
    new GetCallback<List<UserConversation>>() {
    @Override
    public void onSucc(@Nullable List<UserConversation> userConversation) {
        Log.i("MyApplication", "getUserConversation: " + userConversation);
    }

    @Override
    public void onFail(@Nullable String failReason) {
        Log.w("MyApplication", "Failed to getUserConversation: " + failReason);
    }
});
```

### Leaving conversations
To leave a conversation, you can call [`leaveConversation`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#leaveConversation(io.skygear.plugins.chat.Conversation,%20io.skygear.skygear.LambdaResponseHandler)).


```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.leaveConversation(conversation, new LambdaResponseHandler() {
    @Override
    public void onLambdaSuccess(JSONObject result) {
        Log.i("MyApplication", "Left conversation.");
    }

    @Override
    public void onLambdaFail(Error error) {
        Log.i("MyApplication", "Failed to leave conversation: " + error.getLocalizedMessage());
    }
});
```

## Managing conversation participants
At some point of your conversation, you may wish to update the participant list. You may add or remove participants in a conversation.

### Adding users to conversation

You can add users to an existing conversation with [`addConversationParticipant`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#addConversationParticipant(io.skygear.plugins.chat.Conversation,%20java.lang.String,%20io.skygear.plugins.chat.SaveCallback)). In the following code, `userPeter` will be added to the conversation.

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.addConversationParticipant(conversation, userPeter.getId(), new SaveCallback<Conversation>() {
    @Override
    public void onSucc(@Nullable Conversation conversation) {
        Log.i("MyApplication", "Conversation participant added");
    }

    @Override
    public void onFail(@Nullable String failReason) {
        Log.i("MyApplication", "Failed to add participant: " + failReason);
    }
});
```

### Removing users from conversation

To remove users from a conversation, you can call [`removeConversationParticipant`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#removeConversationParticipant(io.skygear.plugins.chat.Conversation,%20java.lang.String,%20io.skygear.plugins.chat.SaveCallback)). In the following code, `userBen` will be removed from the conversation.

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.removeConversationParticipant(conversation, userBen.getId(), new SaveCallback<Conversation>() {
    @Override
    public void onSucc(@Nullable Conversation conversation) {
        Log.i("MyApplication", "Conversation participant removed");
    }

    @Override
    public void onFail(@Nullable String failReason) {
        Log.i("MyApplication", "Failed to remove participant: " + failReason);
    }
});
```

### Admin

An admin of the conversation has the following permissions:
1. add or remove participants from to conversation,
2. add or remove admins from the conversation,
3. delete the conversation

The number of admins in a conversation is unlimited, so you may add everyone as an admin.

#### Adding admins

You can add admins to a conversation by ``:

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.addConversationAdmin(conversation,
    userDavid.getId(),
    new SaveCallback<Conversation>() {
            @Override
            public void onSucc(@Nullable Conversation conversation) {
                Log.i("MyApplication", "Admin set: " + conversation.getId());
            }

            @Override
            public void onFail(@Nullable String failReason) {
                Log.w("MyApplication", "Failed to save: " + failReason);
            }
        });
```

In the above conversation, the current user will be the admin of the conversation.

Example of `setConversationAdminIds`:

In the following example, `userDavid` and `userBen` is added to be the admin of the conversation.

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

HashSet<String> admins = new HashSet<String>();
admin.add(userBen.getId());
admins.add(userDavid.getId());

chatContainer.setConversationAdminIds(conversation,
    admins,
    new SaveCallback<Conversation>() {
        @Override
        public void onSucc(@Nullable Conversation conversation) {
            Log.i("MyApplication", "Admin set: " + conversation.getId());
        }

        @Override
        public void onFail(@Nullable String failReason) {
            Log.w("MyApplication", "Failed to save: " + failReason);
        }
});
```
#### Removing admins
To remove admins from a conversation, you can call [`removeConversationAdmin`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#removeConversationAdmin(io.skygear.plugins.chat.Conversation,%20java.lang.String,%20io.skygear.plugins.chat.SaveCallback)).

## Chat history
### Loading all messages from a conversation
When users get into the chatroom, you may want to load the messages of the conversation. You can specify the limit of the messages in `limit` and the time constraint for the message in `beforeTime`.

You can use [`getMessages`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#getMessages(io.skygear.plugins.chat.Conversation,%20int,%20java.util.Date,%20io.skygear.plugins.chat.GetCallback)) to load all messages in a conversation:

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.getMessages(conversation,
    -1, // -1 as default limit
    null, // Date before
    new GetCallback<List<Message>>() {
    @Override
    public void onSucc(@Nullable List<Message> messageList) {
        Log.i("MyApplication", "Messages Retrieved: " + messageList);
    }

    @Override
    public void onFail(@Nullable String failReason) {
        Log.i("MyApplication", "Failed to get message list: " + failReason);
    }
});
```

## Sending messages

A message in Skygear Chat is a `Message` record.

To send a text message, just specify the `body` of your message and use [`sendMessage`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#sendMessage(io.skygear.plugins.chat.Conversation,%20java.lang.String,%20io.skygear.skygear.Asset,%20org.json.JSONObject,%20io.skygear.plugins.chat.SaveCallback)) to send the message in a conversation.

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.sendMessage(conversation,
                          "Hi! This is a message!",
                          null,
                          null,
                          new SaveCallback<Message>() {
    @Override
    public void onSucc(@Nullable Message message) {
        Log.i("MyApplication", "Message Sent: " + message.getBody());
    }

    @Override
    public void onFail(@Nullable String failReason) {
        Log.i("MyApplication", "Message Failed to be sent: " + failReason);
    }
});
```

### Plain Text

To send a text message, specify the `body` of your message.

You can use the `sendMessage` send a conversation (it works for both direct conversations or group conversations).

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.sendMessage(conversation,
                          "Hi! This is a plain text message!",
                          null,
                          null,
                          new SaveCallback<Message>() {
    @Override
    public void onSucc(@Nullable Message message) {
        Log.i("MyApplication", "Message Sent: " + message.getBody());
    }

    @Override
    public void onFail(@Nullable String failReason) {
        Log.i("MyApplication", "Message Failed to be sent: " + failReason);
    }
});

```

### Metadata

Besides the body of the message, you may wish to specify metadata in your message. For example, special format or color of your message.

`metadata` can contain a JSON format object

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

JSONObject meta = new JSONObject();
meta.put("textColor", "red"); //custom metadata
meta.put("isImportant",new Boolean(true)); //custom metadata

chatContainer.sendMessage(conversation, "Message Body", null, meta, new SaveCallback<Message>() {
    @Override
    public void onSucc(@Nullable Message message) {
        Log.i("MyApplication", "Message Sent: " + message.getBody());
    }

    @Override
    public void onFail(@Nullable String failReason) {
        Log.i("MyApplication", "Message Failed to be sent: " + failReason);
    }
});
```
### Files

If you would like to send files via Skygear Chat, you can upload a file as an [`io.skygear.skygear.Asset`](https://docs.skygear.io/android/reference/io/skygear/skygear/Asset.html).

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

Asset asset = new Asset("filename", sourceUrl);

chatContainer.sendMessage(conversation,
                          "Hi! This is a messagewith assets",
                          null,
                          asset,
                          new SaveCallback<Message>() {
    @Override
    public void onSucc(@Nullable Message message) {
        Log.i("MyApplication", "Message Sent: " + message.getBody());
        // You can call message.getAsset() to get the asset
    }

    @Override
    public void onFail(@Nullable String failReason) {
        Log.i("MyApplication", "Message Failed to be sent: " + failReason);
    }
});
```


## Subscribing to new messages
### Subscribing to messages in a conversation

In order to get real time update of new messages, you can subscribe to a conversation with [`subscribeConversationMessage`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#subscribeConversationMessage(io.skygear.plugins.chat.Conversation,%20io.skygear.plugins.chat.MessageSubscriptionCallback)).

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.subscribeConversationMessage(conversation, new MessageSubscriptionCallback(conversation) {
    @Override
    public void notify(@NonNull String eventType, @NonNull Message message) {
        Log.i("MyApplication", "Event Type:"+ eventType +", Message: " + message.getBody());
    }

    @Override
    public void onSubscriptionFail(@Nullable String reason) {
        Log.i("MyApplication", "subscription Failed to be sent: " + reason);
    }
});
```
To unsubscribe from a conversation, simply call [`unsubscribeConversationMessage`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#unsubscribeConversationMessage(io.skygear.plugins.chat.Conversation)).

### Subscribing to messages in all conversations

Besides a specific conversation, you might want to get notified whenever there are new messages in any conversation you belong to.

- Coming Soon

## Displaying unread count

### Conversation unread count
You can show the unread count for different conversations in the conversation list with [`getConversationUnreadMessageCount`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#getConversationUnreadMessageCount(io.skygear.plugins.chat.Conversation,%20io.skygear.plugins.chat.GetCallback)).

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.getConversationUnreadMessageCount(conversation, new GetCallback<Integer>() {
    @Override
    public void onSucc(@Nullable Integer count) {
        Log.i("MyApplication", "There are " + count + " unreads in " + conversation.getTitle());
    }

    @Override
    public void onFail(@Nullable String failReason) {
        Log.i("MyApplication", "Failed to get unread count: " + failReason);
    }
});
```

### Overall unread count
You may wish to show the overall unread count of all conversations in the badge value of your app with [`getTotalUnreadMessageCount`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#getTotalUnreadMessageCount(io.skygear.plugins.chat.GetCallback)).

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.getTotalUnreadMessageCount(new GetCallback<Integer>() {
    @Override
    public void onSucc(@Nullable Integer count) {
        Log.i("MyApplication", "There are total " + count + " unreads");
    }

    @Override
    public void onFail(@Nullable String failReason) {
        Log.i("MyApplication", "Failed to get unread count: " + failReason);
    }
});
```

### Resetting the unread count
The unread count can be reset by [marking the messages as read](#marking-messages-as-read).

## Typing indicator
The [`Typing.State`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/Typing.State.html) has these events:

- `BEGIN` - User began typing
- `PAUSE` - User stopped typing
- `FINISH`- User stopped typing and the message is sent

You can make good use of these events to implement the typing indicator feature in your app.

### Subscribing to typing indicator
Skygear Chat provides real-time update to typing indicators in a particular conversation.
with [`subscribeTypingIndicator`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#subscribeTypingIndicator(io.skygear.plugins.chat.Conversation,%20io.skygear.plugins.chat.TypingSubscriptionCallback)).

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.subscribeTypingIndicator(conversation, new TypingSubscriptionCallback(conversation) {
    @Override
    public void notify(@NonNull Map<String, Typing> typingMap) {
    // Handle event here
    }

    @Override
    public void onSubscriptionFail(@Nullable String reason) {
        Log.i("MyApplication", "Failed to subscribe: " + reason);
    }
});
```

### Sending my typing status
To get typing status from other devices, you should always update your typing status to the server with [`sendTypingIndicator`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/ChatContainer.html#sendTypingIndicator(io.skygear.plugins.chat.Conversation,%20io.skygear.plugins.chat.Typing.State)).

```Java
chatContainer.sendTypingIndicator(conversation, Typing.State.BEGIN);
```
```Java
chatContainer.sendTypingIndicator(conversation, Typing.State.PAUSE);
```
```Java
chatContainer.sendTypingIndicator(conversation, Typing.State.FINISH);
```

## Recipient status
Skygear Chat is helpful for displaying recipient status such as *"Delivering"*, *"Delivered"* or *"Seen"*.

### Message status
You can make use of the following receipt status to indicate your message status.

List of [`Message.Status`](https://docs.skygear.io/android/plugins/chat/reference/io/skygear/plugins/chat/Message.Status.html) status:
- `DELIVERED` - The message is delivered, but it is not read yet.
- `SOME_READ` - The message is delivered and read by some participants.
- `ALL_READ` - The message is delivered and read by all.

### Subscribing to message status change

By subscribing to `Message.Status` in a message, you can get the latest status of the message sent to other recipients.

```Java
switch(message.getStatus()){
    case ALL_READ:
        Log.i("Message", "All read");
        break;
    case SOME_READ:
        Log.i("Message", "Some read");
        break;
    case DELIVERED:
        Log.i("Message", "Delivered");
        break;
    default:
        Log.i("Message", "Delivering");
}
```

### Marking messages as read

On the recipient client side, you need to update the message status if the message is read. For example, in `onStart` method of your message view activity.

```Java
chatContainer.markMessagesAsRead(messages);
```

Note: `messages` is a List of `Message`.


You can also mark a last read message in the conversation:

```Java
chatContainer.markConversationLastReadMessage(conversation, message);
```

## Push notification
### Sending push notifications to conversation participants

We can send the push notification to particular `userIds` and these can be retrieved by accessing the attribute `participantsIds` of `Conversation`.

```Java
// Coming soon
```

### Receiving push notifications
Coming soon

## User online
You can display whether the user is online or the last seen time accordingly.

### User online indicator status
Coming soon

### Subscribing to user online indicator
Coming soon

## Best practices
### Caching message history locally

Skygear Chat does not have support on caching message history locally. However you can always achieve offline caching with other tools.

There are some good libraries helping you to cache messages offline:
- [`Android dualcache`](https://github.com/vincentbrison/dualcache)
- [`Disk LRU Cache`](https://github.com/JakeWharton/DiskLruCache)

### Handling edit and delete messages

You can edit or delete `Message` records like other records. Then save it to the cloud.

Here is an example on how you can edit the body text of the last message in a conversation:


```Java
Record messageRecord = message.getRecord();
messageRecord.set("body","Updated body content");

try {
    skygear.getPrivateDatabase().save(messageRecord, new RecordSaveResponseHandler() {
        @Override
        public void onSaveSuccess(Record[] records) {
            Log.i("MyApplication", "Message body saved");
        }

        @Override
        public void onPartiallySaveSuccess(Map<String, Record> successRecords, Map<String, Error> errors) {
            Log.i("MyApplication", "Some messages failed to save");
        }

        @Override
        public void onSaveFail(Error error) {
            Log.w("MyApplication", "Failed to save message: " + error.getMessage(), error);
        }
    });
} catch (AuthenticationException e) {
    e.printStackTrace();
}

```
