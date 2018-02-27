---
title: Skygear Chat
---

[[toc]]

# Skygear Chat Basic (Android)
Skygear Chat is a collection of APIs to help you build Chat apps much easier.

Skygear Chat is built on top of the cloud database and the PubSub module. If you want to know more about how the underlying works, check out the guide for [cloud database](https://docs.skygear.io/guides/cloud-db/basics/android/) and [PubSub](https://docs.skygear.io/guides/pubsub/basics/android/).


## Enabling Chat
To start using Skygear Chat feature, make sure you have already enabled Chat in the Plug-ins tab in your [Skygear Portal] (https://portal.skygear.io). Please refer to [Quick Start](https://docs.skygear.io/guides/chat/quick-start/android/) for the instructions.

## Users
Most real-time chat apps require a user login system and you can do it with the Skygear user authentication module. It offers various sign up methods - sign up with email, sign up with username and sign up with Facebook and Google account.

Check out the guide for [User Authentication Basics](https://docs.skygear.io/guides/auth/basics/android/) and [User Profile Best Practices](https://docs.skygear.io/guides/auth/user-profile/android/).

## Conversations
In order to send messages, you need to create a conversation first. You can consider **conversations** as chatrooms or channels in your application.

There are two types of conversations in Skygear:
- **Direct Conversation**, support chatting between one user to another
- **Group Conversation**, support chatting among 2 or more users

### Creating direct conversations

You can use [`createDirectConversation`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#createDirectConversation-java.lang.String-java.lang.String-java.util.Map-io.skygear.plugins.chat.SaveCallback-) to create a conversation with another user. Please specify the user ID as `userID`.

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.createDirectConversation(userBen.getId(), "Chat with Ben", null, new SaveCallback<Conversation>() {
    @Override
    public void onSuccess(@Nullable Conversation conversation) {
        Log.i("MyApplication", "Created: " + conversation.getId());
    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.w("MyApplication", "Failed to save: " + error.getMessage());
    }
});
```

This example shows how to create a direct chat between the current user and `userBen`.

You can also set up optional `metadata` in your conversation for custom configurations.

### Creating group conversations
Besides direct chats, you can also create a group conversation with 2 or more people.

Instead of a single `userID` as the parameter, [`createConversation`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#createConversation-java.util.Set-java.lang.String-java.util.Map-java.util.Map-io.skygear.plugins.chat.SaveCallback-) accepts a list of `userIDs` in a Set of String . A new conversation will be created with the IDs given as participants.

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

HashSet<String> users = new HashSet<String>();
users.add(skygear.getAuth().getCurrentUser().getId());
users.add(userBen.getId());
users.add(userChris.getId());

chatContainer.createConversation(users, "Random Conversation", null, null, new SaveCallback<Conversation>() {
    @Override
    public void onSuccess(@Nullable Conversation conversation) {
        Log.i("MyApplication", "Created: " + conversation.getId());

    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.w("MyApplication", "Failed to save: " + error.getMessage());
    }
});
```

### Creating group chat by distinct participants *(DISTINCT\_BY\_PARTICIPANTS)*

By default, if you try to create conversations with same list of participants with [`createConversation`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#createConversation-java.util.Set-java.lang.String-java.util.Map-java.util.Map-io.skygear.plugins.chat.SaveCallback-). You will eventually create different conversations with the identical participants.

This may or may not be a desire behavior in your application depends on your app design.


In the options, you can pass in [`DISTINCT_BY_PARTICIPANTS`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Conversation.OptionKey.html#DISTINCT_BY_PARTICIPANTS) to set the conversations to be distinct by participants.

By default, [`DISTINCT_BY_PARTICIPANTS`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Conversation.OptionKey.html#DISTINCT_BY_PARTICIPANTS) is set to be `false`.

You can also set it with a separated call on  [`setConversationDistinctByParticipants`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#setConversationDistinctByParticipants-io.skygear.plugins.chat.Conversation-boolean-io.skygear.plugins.chat.SaveCallback-).

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);
String conversationTitle = "Greeting";

HashSet<String> participants = new HashSet<String>();
participants.add(userBen.getId());
participants.add(userCharles.getId());
participants.add(userDavid.getId());
participants.add(userEllen.getId());

Map<Conversation.OptionKey, Object> options = new HashMap<>();
options.put(Conversation.OptionKey.DISTINCT_BY_PARTICIPANTS, true);

chatContainer.createConversation(participants, conversationTitle null, options, new SaveCallback<Conversation>() {
    @Override
    public void onSuccess(@Nullable Conversation conversation) {
        Log.i("MyApplication", "Created: " + conversation.getId());

    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.w("MyApplication", "Failed to save: " + error.getMessage());
    }
});
```

Since we have set [`DISTINCT_BY_PARTICIPANTS`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Conversation.OptionKey.html#DISTINCT_BY_PARTICIPANTS) to be true, upon calling the above function twice, you will receive an error as duplicated conversations.

If the conversation already exists, then `onFail(@NonNull Error error)` callback returns a `ConversationAlreadyExistsError` error object. You can retrieve the original conversation ID via casting `error` object to  a `ConversationAlreadyExistsError` object and calling `getConversationId()` method.

```Java
chatContainer.createConversation(participants, conversationTitle null, options, new SaveCallback<Conversation>() {
    @Override
    public void onSuccess(@Nullable Conversation conversation) {
        Log.i("MyApplication", "Created: " + conversation.getId());

    }

    @Override
    public void onFail(@NonNull Error error) {
        if (error instanceof ConversationAlreadyExistsError) {
            ConversationAlreadyExistsError alreadyExistsError = (ConversationAlreadyExistsError) error;
            Log.w("MyApplication", "Conversation Already Exists " + alreadyExistsError.getConversationId());
        } else {
            Log.w("MyApplication", "Failed to save: " + error.getMessage());
        }
    }
});
```

Note: `DISTINCT_BY_PARTICIPANTS ` will be `false` automatically after participant list is being altered.

There are a few more attributes you can specify in [`createConversation`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#createConversation-java.util.Set-java.lang.String-java.util.Map-java.util.Map-io.skygear.plugins.chat.SaveCallback-) :

- **title**: That's the title of a conversation. It can be omitted.
- **metadata**: It is for application specific purpose. Say in your app you have different type of conversations, you can specific it here.

### Setting admins

All users in `participantIDs` will be administrators unless [`ADMIN_IDS`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Conversation.OptionKey.html#ADMIN_IDS) is specified in [`createConversation`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#createConversation-java.util.Set-java.lang.String-java.util.Map-java.util.Map-io.skygear.plugins.chat.SaveCallback-).


```Java
HashSet<String> admins = new HashSet<String>();
admins.add(userBen.getId());

Map<Conversation.OptionKey, Object> options = new HashMap<>();
options.put(Conversation.OptionKey.ADMIN_IDS, admins);

chatContainer.createConversation(participants, conversationTitle, null, options, new SaveCallback<Conversation>() {
    @Override
    public void onSuccess(@Nullable Conversation conversation) {
        Log.i("MyApplication", "Created: " + conversation.getId());

    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.w("MyApplication", "Failed to save: " + error.getMessage());
    }
});
```

In the above example, `userBen` will be the only administrator.

### Fetching existing conversations

In `io.skygear.plugins.chat`, [`Conversation`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Conversation.html) represents a conversation, it contains information of a conversation that is shared among all participants, such as `participantIds` and `adminIds`.

There are two methods to fetch existing conversations:

- [`getConversations`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#getConversations-io.skygear.plugins.chat.GetCallback-)
- [`getConversation`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#getConversation-java.lang.String-io.skygear.plugins.chat.GetCallback-)

Fetch all conversations which involves the current user. The parameter `getLastMessage` can be used for fetching the last message and last read message in the conversation.


```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.getConversations(new GetCallback<List<Conversation>>() {
    @Override
    public void onSuccess(List<Conversation> result) {
        Log.i("MyApplication", "Fetched " + result.count() + " conversations.");
    }

    @Override
    public void onFail(String error.getMessage()) {
        Log.i("MyApplication", "Failed to getConversations: " + error.getLocalizedMessage());
    }
});
```

### Updating a conversation
Conversation can be updated via [`setConversationMetadata`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#setConversationMetadata-io.skygear.plugins.chat.Conversation-java.util.Map-io.skygear.plugins.chat.SaveCallback-) and [`setConversationTitle`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#setConversationTitle-io.skygear.plugins.chat.Conversation-java.lang.String-io.skygear.plugins.chat.SaveCallback-).

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.setConversationTitle(conversation, "New Title", new SaveCallback<Conversation>() {
    @Override
    public void onSuccess(Conversation result) {
        Log.i("MyApplication", "Conversation Updated.");
    }

    @Override
    public void onFail(String error.getMessage()) {
        Log.i("MyApplication", "Failed to setConversationTitle: " + error.getLocalizedMessage());
    }
});
```

### Leaving conversations
To leave a conversation, you can call [`leaveConversation`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#leaveConversation-io.skygear.plugins.chat.Conversation-LambdaResponseHandler-).


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

You can add users to an existing conversation with [`addConversationParticipant`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#addConversationParticipant-io.skygear.plugins.chat.Conversation-java.lang.String-io.skygear.plugins.chat.SaveCallback-). In the following code, `userPeter` will be added to the conversation.

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.addConversationParticipant(conversation, userPeter.getId(), new SaveCallback<Conversation>() {
    @Override
    public void onSuccess(@Nullable Conversation conversation) {
        Log.i("MyApplication", "Conversation participant added");
    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.i("MyApplication", "Failed to add participant: " + error.getMessage());
    }
});
```

### Removing users from conversation

To remove users from a conversation, you can call [`removeConversationParticipant`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#removeConversationParticipant-io.skygear.plugins.chat.Conversation-java.lang.String-io.skygear.plugins.chat.SaveCallback-). In the following code, `userBen` will be removed from the conversation.

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.removeConversationParticipant(conversation, userBen.getId(), new SaveCallback<Conversation>() {
    @Override
    public void onSuccess(@Nullable Conversation conversation) {
        Log.i("MyApplication", "Conversation participant removed");
    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.i("MyApplication", "Failed to remove participant: " + error.getMessage());
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

You can add admins to a conversation by [`addConversationAdmin `](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#addConversationAdmin-io.skygear.plugins.chat.Conversation-java.lang.String-io.skygear.plugins.chat.SaveCallback-):

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.addConversationAdmin(conversation,
    userDavid.getId(),
    new SaveCallback<Conversation>() {
            @Override
            public void onSuccess(@Nullable Conversation conversation) {
                Log.i("MyApplication", "Admin set: " + conversation.getId());
            }

            @Override
            public void onFail(@NonNull Error error) {
                Log.w("MyApplication", "Failed to save: " + error.getMessage());
            }
        });
```

In the above conversation David will be the admin of the conversation.

#### Removing admins
To remove admins from a conversation, you can call [`removeConversationAdmin`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#removeConversationAdmin-io.skygear.plugins.chat.Conversation-java.lang.String-io.skygear.plugins.chat.SaveCallback-).

## Messages
Skygear Chat supports real time messaging. A message is the real content of a conversation. Skygear Chat supports 2 types of messages, one is plain text, the other one is assets. Assets include files, images, voice message and video.

### Loading messages from a conversation
When users get into the chatroom, you may call [`getMessages`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#getMessages-io.skygear.plugins.chat.Conversation-int-java.util.Date-java.lang.String-io.skygear.plugins.chat.GetCallback-) to load the messages of the conversation. You can specify the limit of the messages in `limit` and the time constraint for the message in `before`.

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.getMessages(conversation,
    -1, // -1 as default limit
    null, // Date before
    new GetMessagesCallback() {
    @Override
    public void onSuccess(@Nullable List<Message> messageList) {
        Log.i("MyApplication", "Messages Retrieved from server: " + messageList);
    }

    @Override
    public void onGetCachedResult(@Nullable List<Message> messageList) {
        Log.i("MyApplication", "Messages Retrieved from cache: " + messageList);
    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.i("MyApplication", "Failed to get message list: " + error.getMessage());
    }
});
```

### Sending messages

A message in Skygear Chat is a `Message` record.

To send a text message, just specify the `body` of your message and use [`sendMessage`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#sendMessage-io.skygear.plugins.chat.Conversation-java.lang.String-Asset-JSONObject-io.skygear.plugins.chat.SaveCallback-) to send the message in a conversation.

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.sendMessage(conversation,
                          "Hello!",
                          null,
                          null,
                          new SaveCallback<Message>() {
    @Override
    public void onSuccess(@Nullable Message message) {
        Log.i("MyApplication", "Message Sent: " + message.getBody());
    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.i("MyApplication", "Message Failed to be sent: " + error.getMessage());
    }
});
```


When adding a message to a conversation, the operation is created and is added
to the local cache store. The callback function is called
when the message is added to server.

#### Handling failed messages

If messages failed to save, you can obtain them from
`fetchOutstandingMessageOperations`.

Outstanding message operations contain messages that are not yet synchronized
to the server. You can fetch outstanding message operations to display to
the user which messages are failed to synchronize, so that you can present
user with options whether to retry or cancel the operation.

Here is an example:

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.fetchOutstandingMessageOperations(conversation, MessageOperation.Type.ADD, new GetCallback<List<MessageOperation>>() {
    @Override
    public void onSuccess(@Nullable List<MessageOperation> operations) {
        Log.i("MyApplication", "Outstanding Message Operation Retrieved: " + operations);
    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.i("MyApplication", "Failed to get outstanding message operations: " + error.getMessage());
    }
});
```

To retry a message operation, calls `retryMessageOperation`:

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.fetchOutstandingMessageOperations(message, MessageOperation.Type.ADD, new GetCallback<List<MessageOperation>>() {
    @Override
    public void onSuccess(@Nullable List<MessageOperationOperation> operations) {
        for (MessageOperation operation : operations) {
            chatContainer.retryMessageOperation(operation, new MessageOperationCallback() {
                @Override
                public void onSuccess(MessageOperation operation, Message message) {
                    Log.i("MyApplication", "Retried message operation: " + operation);
                }

                @Override
                public void onFail(@NonNull Error error) {
                    Log.i("MyApplication", "Failed to retry outstanding message operation: " + error.getMessage());
                }
            });
        }
    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.i("MyApplication", "Failed to fetch outstanding message operations: " + error.getMessage());
    }
});
```

Calls `cancelMessageOperation` to cancel the message operation instead.

#### Plain Text

To send a text message, specify the `body` of your message.

You can use the [`sendMessage`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#sendMessage-io.skygear.plugins.chat.Conversation-java.lang.String-Asset-JSONObject-io.skygear.plugins.chat.SaveCallback-) send a conversation (it works for both direct conversations or group conversations).

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.sendMessage(conversation,
                          "Hi! This is a plain text message!",
                          null,
                          null,
                          new SaveCallback<Message>() {
    @Override
    public void onSuccess(@Nullable Message message) {
        Log.i("MyApplication", "Message Sent: " + message.getBody());
    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.i("MyApplication", "Message Failed to be sent: " + error.getMessage());
    }
});

```

#### Metadata

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
    public void onSuccess(@Nullable Message message) {
        Log.i("MyApplication", "Message Sent: " + message.getBody());
    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.i("MyApplication", "Message Failed to be sent: " + error.getMessage());
    }
});
```
#### Files

If you would like to send files via Skygear Chat, you can upload a file as an [`io.skygear.skygear.Asset`](https://docs.skygear.io/android/reference/latest/io/skygear/skygear/Asset.html).

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

Asset asset = new Asset("filename", sourceUrl);

chatContainer.sendMessage(conversation,
                          "Hi! This is a message with assets",
                          null,
                          asset,
                          new SaveCallback<Message>() {
    @Override
    public void onSuccess(@Nullable Message message) {
        Log.i("MyApplication", "Message Sent: " + message.getBody());
        // You can call message.getAsset() to get the asset
    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.i("MyApplication", "Message Failed to be sent: " + error.getMessage());
    }
});
```

### Editing a message
You can edit a message with [`editMessage`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#editMessage-io.skygear.plugins.chat.Message-java.lang.String-io.skygear.plugins.chat.SaveCallback-).

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);
chatContainer.editMessage(message, "New Body", new SaveCallback<Message>() {
    @Override
    public void onSuccess(@Nullable Message message) {
        Log.i("MyApplication", "Message Updated: " + message.body);
    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.i("MyApplication", "Cannot edit message: " + error.getMessage());
    }
});
```

### Deleting a message
You can delete a message with [`deleteMessage`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#deleteMessage-io.skygear.plugins.chat.Message-io.skygear.plugins.chat.DeleteCallback-).

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);
chatContainer.deleteMessage(message, new DeleteCallback<Message>() {
    @Override
    public void onSuccess(@Nullable Message message) {
        Log.i("MyApplication", "Message deleted.");
    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.i("MyApplication", "Cannot delete message: " + error.getMessage());
    }
});
```

## Subscribing to new messages
### Subscribing to messages in a conversation

In order to get real time update of new messages, you can subscribe to a conversation with [`subscribeConversationMessage`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#unsubscribeConversationMessage-io.skygear.plugins.chat.Conversation-).

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
To unsubscribe from a conversation, simply call [`unsubscribeConversationMessage`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#unsubscribeConversationMessage-io.skygear.plugins.chat.Conversation-).

### Subscribing to messages in all conversations

- Coming Soon

### Callback data
The callback passes a data object as follows:

```javascript!
{
  "record_type": "message",
  "event_type": "create",
  "record": recordObj,
  "original_record": nulll
}
```

The event_type may contain the following strings:

- create - new message received from others, and it should be inserted to your conversation UI
- update - when a message updated, or if the delivery or read status change (e.g. from `delivered` to `some_read` at `message_status`)
- delete - when a message was deleted

## Push notification
### Sending push notifications to conversation participants

We can send the push notification to particular `userIds` and these can be retrieved by accessing the attribute `participantsIds` of [`Conversation`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Conversation.html).

Coming soon

### Receiving push notifications
Coming soon

## Receipt
Skygear Chat provides you with message receipt which includes status and timestamps information.

### Message status
You can make use of the following receipt status to indicate your message status.

### Message status
You can make use of the following receipt status to indicate your message status.

[`Message.Status`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Message.Status.html) has 3 values.

| Status  | Description  |
| ------  | ------------ |
|[`DELIVERED`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Message.Status.html#DELIVERED)|The message is delivered, but it is not read yet.|
|[`SOME_READ`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Message.Status.html#SOME_READ)|The message is delivered and read by some participants.|
|[`ALL_READ`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Message.Status.html#ALL_READ)|The message is delivered and read by all participants.|

### Subscribing to message status change

By subscribing to [`Message.Status`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Message.Status.html) in a message, you can get the latest status of the message sent to other recipients.

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

On the recipient client side, you need to update the message status if the message is read. For example, in [`onStart`](https://developer.android.com/reference/android/app/Activity.html#onStart()) method of your message view activity.

```Java
chatContainer.markMessagesAsRead(messages);
```

Note: `messages` is a List of [`Message`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Message.html).


## Message unread count

### Conversation unread count
You can show the unread count of a [`Conversation`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Conversation.html) with [`getUnreadCount()`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Conversation.html#getUnreadCount--).

### Total unread count
You may wish to show the overall unread count of all conversations in the badge value of your app with [`getTotalUnreadMessageCount`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#getTotalUnreadMessageCount-io.skygear.plugins.chat.GetCallback-).

```Java
Container skygear = Container.defaultContainer(getApplicationContext());
ChatContainer chatContainer = ChatContainer.getInstance(skygear);

chatContainer.getTotalUnreadMessageCount(new GetCallback<Integer>() {
    @Override
    public void onSuccess(@Nullable Integer count) {
        Log.i("MyApplication", "There are total " + count + " unreads");
    }

    @Override
    public void onFail(@NonNull Error error) {
        Log.i("MyApplication", "Failed to get unread count: " + error.getMessage());
    }
});
```

### Resetting the unread count
The unread count can be reset by [marking the messages as read](#marking-messages-as-read).

## Typing indicator
The [`Typing.State`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Typing.State.html) has these events:

- [`BEGIN`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Typing.State.html#BEGIN) - User began typing
- [`PAUSE`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Typing.State.html#PAUSE) - User stopped typing
- [`FINISH`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/Typing.State.html#FINISHED)- User stopped typing and the message is sent

You can make good use of these events to implement the typing indicator feature in your app.

### Subscribing to typing indicator
Skygear Chat provides real-time update to typing indicators in a particular conversation.
with [`subscribeTypingIndicator`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#subscribeTypingIndicator-io.skygear.plugins.chat.Conversation-io.skygear.plugins.chat.TypingSubscriptionCallback-).

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

### Sending an User's typing status
To get typing status from other devices, you should always update your typing status to the server with [`sendTypingIndicator`](https://docs.skygear.io/android/chat/reference/latest/io/skygear/plugins/chat/ChatContainer.html#sendTypingIndicator-io.skygear.plugins.chat.Conversation-io.skygear.plugins.chat.Typing.State-).

```Java
chatContainer.sendTypingIndicator(conversation, Typing.State.BEGIN);
```
```Java
chatContainer.sendTypingIndicator(conversation, Typing.State.PAUSE);
```
```Java
chatContainer.sendTypingIndicator(conversation, Typing.State.FINISH);
```


## User online
Coming soon
