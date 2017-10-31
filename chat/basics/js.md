---
title: Skygear Chat
---

[[toc]]

# Skygear Chat Basic (JavaScript)
Skygear Chat is a collection of APIs to help you build real-time chat apps a lot easier.

Skygear Chat is built on top of the cloud database and the PubSub module. If you want to know more about how the underlying works, check out the guide for [cloud database](https://docs.skygear.io/guides/cloud-db/basics/js/) and [PubSub](https://docs.skygear.io/guides/pubsub/basics/js/).

## Enabling Chat
To start using Skygear Chat feature, make sure you have already enabled Chat in the Plug-ins tab in your [Skygear Portal] (https://portal.skygear.io). Please refer to [Quick Start](https://docs.skygear.io/guides/chat/quick-start/js/) for the instructions.

## Users
Most real-time chat apps require a user login system and you can do it with the Skygear user authentication module. It offers various sign up methods - sign up with email, sign up with username and sign up with Facebook and Google account.

Check out the guide for [User Authentication Basics](https://docs.skygear.io/guides/auth/basics/js/) and [User Profile Best Practices](https://docs.skygear.io/guides/auth/user-profile/js/).

## Conversations

In order to send messages, you need to create a conversation first. You can consider conversations as chatrooms or channels in your application.

There are two types of conversations in Skygear Chat:

- **Direct Conversation**, a chatting group between one and another user
- **Group Conversation**,  a chatting group among 2 or more users

### Creating direct conversations 

You can use [`createDirectConversation`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-createDirectConversation) to create a conversation with another user. Please specify the user ID as `userID`. 

```javascript
skygearChat.createDirectConversation(userBen, 'Greeting')
  .then(function (conversation) {
    // a new direct conversation is created
    console.log('Conversation created!', conversation);
  }, function (err) {
    console.log('Failed to create conversation');
  });
```

The above example creates a direct chat between the current user and `userBen`.

You can also set up an optional parameter, `metadata`, for customization.
  
### Creating group conversations

Besides direct chats, you can also create a group conversation with 2 or more people. 

Instead of a single `userID` parameter, [`createConversation`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-createConversation) accepts a list of users called `participantIDs`. A new conversation will be created with the given participant IDs.

```javascript!
const participants = [userBen, userCharles, userDavid, userEllen];

skygearChat.createConversation(participants, 'Greeting')
  .then(function (conversation) {
    // a new group conversation is created
    console.log('Conversation created!', conversation);
  }, function (err) {
    console.log('Failed to create conversation');
  });
```

### Creating group chat by distinct participants (`distinctByParticipants`)

By default, [`createConversation`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-createConversation) allows you to create multiple conversations with identical participants. However, depending on application design, it may not be a desired behavior. Therefore, Skygear Chat provides `distinctByParticipants` option, which enables distinctive participants in every conversation.

The default value of `distinctByParticipants` is `false`. If it is set to be `true`, then the participants cannot be identical to any existing conversation. Otherwise, an error will be returned.

```Javascript!
const participants = [userBen, userCharles, userDavid, userEllen];
const conversationTitle = "Greeting";
const conversationMeta = {};
const conversationOptions = {
    distinctByParticipants: true
};

skygearChat.createConversation(participants, conversationTitle, conversationMeta , conversationOptions)
  .then(function (conversation) {
    // a group conversation is created

    console.log('Conversation created!', conversation);
  }, function (err) {
    console.log('Failed to create conversation');
  });
```

Note: `distinctByParticipants` will be `false` automatically after participant list is being altered.

There are a few more attributes you can specify when you create a conversation:
`skygearChat.createConversation(partcipants, title, meta, options)`.

- **title**: That's the title of a conversation. It can be omitted.
- **meta**: It is for application specific purpose. Say in your app you have different type of conversations, you can specific it here.


### Setting admins
All users in `participantIDs` will be administrators unless `adminIDs` is specified in [`createConversation`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-createConversation).


```JavaScript!
const participants = [userBen, userCharles, userDavid, userEllen];
const conversationTitle = "Greeting";

skygearChat.createConversation(participants, conversationTitle, {}, {admins: [userBen]})
  .then(function (conversation) {
    // a new conservation is created
    // Ben will be the only admin of the conversation

    console.log('Conversation created!', conversation);
  }, function (err) {
    console.log('Failed to create conversation');
  });
```

In the above example, `userBen` will be the only administrator.

### Fetching existing conversations

In `skygear-chat`, `Conversation` represents a conversation, it contains information of a conversation that is shared among all participants, such as `participantIds` and `adminIds`.

There are two methods to fetch existing conversations:

- [`getConversations`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-getConversations)
- [`getConversation`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-getConversation)

Fetch all conversations which involves the current user. The parameter `fetchLastMessage` can be used for fetching the last message and last read message in the conversation.

```JavaScript!
skygearChat.getConversations().then(function (conversations) {
  console.log('Fetched ' + conversations.length.toString() + 'conversations.');
});
```

Please refer to the documentation for more options [here](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-getConversations).


### Updating a conversation
Conversation can be updated via [`updateConversation`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-updateConversation).

```javascript!
skygearChat.updateConversation(conversation, "new title", null)
  .then(function(conversation) {
    console.log('Conversation updated');
  }, function (err) {
    console.log('Failed to update the conversation');
  });
```


### Leaving a conversation

To leave a conversation, you can call [`leaveConversation`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js%7ESkygearChatContainer.html#instance-method-leaveConversation).

```javascript!
skygearChat.leaveConversation(conversation)
  .then(function (conversation) {
    // you have left the conversation

    console.log('Left conversation', conversation);
  }, function (err) {
    console.log('Failed to leave the conversation');
  });
```

## Managing conversation participants
At some point of your conversation, you may wish to update the participant list. You may add or remove participants in a conversation.

### Adding users to conversation 

You can add users to an existing conversation with [`addParticipants`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-addParticipants). In the following code, `userBen`, `userCharles`, `userDavid` and `userEllen` will be added to `conversation`.

```JavaScript!
skygearChat.addParticipants(conversation, [userBen, userCharles, userDavid, userEllen])
  .then(function (conversation) {
    console.log('Users added to the conversation.');
  });                   
});
```

### Removing users from conversation 

To remove users from a conversation, you can call [`removeParticipants`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-removeParticipants). In the following code, `userBen` and `userCharles` will be removed from `conversation`.

```JavaScript!
skygearChat.removeParticipants(conversation, [userBen, userCharles])
  .then(function (conversation) {
    console.log('Users removed from the conversation.');
  });                   
});
```


### Admin 

An admin of the conversation has the following permissions:

1. add or remove participants from to conversation, 
2. add or remove admins from the conversation and
3. delete the conversation

The number of admins in a conversation is unlimited, so you may add everyone as an admin.

#### Adding admins

You can add admins to a conversation via calling [`addAdmins`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-addAdmins). In the following example, `userDavid` and `userEllen` are added as admins of the conversation.

```JavaScript!
skygearChat.addAdmins(conversation, [userDavid, userEllen])
  .then(function (conversation) {
    console.log('Admins added to the conversation.');
  });                   
});
```

#### Removing admins
To remove admins from a conversation, use [`removeAdmins`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-removeAdmins). In the following example, `userDavid` and `userEllen` are no longer admins.

```JavaScript!
skygearChat.removeAdmins(conversation, [userDavid, userEllen])
  .then(function (conversation) {
    console.log('Admins removed from the conversation.');
  });                   
});
```


## Messages
Skygear Chat supports real time messaging. A message is the real content of a conversation. Skygear Chat supports 2 types of messages, one is plain text, the other one is assets. Assets include files, images, voice message and video.

### Loading messages from a conversation 
When users get into the chatroom, you may call [`getMessages`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-getMessages) to load the messages of the conversation. You can specify the limit of the messages in `limit` and the time constraint for the message in `beforeTime`.


```JavaScript
const limit=10;
const beforeTime = new Date();

skygearChat.getMessages(conversation, limit, beforeTime)
  .then(function (messages) {
    // messages contains the latest 10 messages
  });
```

### Sending messages

To send a message, you can call [`createMessage`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-createMessage) and specify a target conversation.

This is an example of sending a red plain text message together with a file.

```javascript!
const message = "hello!";

skygearChat.createMessage(conversation, message, null, null)
  .then(function (result) {
    console.log('Message sent!', result);
});
```

#### Plain Text

To send a text message, specify the `body` of your message.

You can use the [`createMessage`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-createMessage) send a conversation (it works for both direct conversations or group conversations).

```javascript!
const message = "Hi! This is a plain text message!";

skygearChat.createMessage(conversation, message, null, null)
  .then(function (result) {
    console.log('Message sent!', result);
});
```

#### Metadata

Besides the body of the message, you may wish to specify metadata in your message. For example, special format or color of your message.

`metadata` can contain a JSON format object

```javascript!
const message = "HMessage Body";
const metadata = {
    textColor: 'red',
    isImportant: true
};


skygearChat.createMessage(conversation, message, metadata, null)
  .then(function (result) {
    console.log('Message sent!', result);
});
```
#### Files

If you would like to send files via Skygear Chat, you can upload a file as an [`Asset`](https://docs.skygear.io/js/reference/latest/class/packages/skygear-core/lib/asset.js~Asset.html).

```javascript!
const message = "Hi! This is a message with assets";
// Getting the asset file using jQuery
const assets = $('message-asset').files[0];

skygearChat.createMessage(conversation, message, null, assets)
  .then(function (result) {
    console.log('Message sent!', result);
});
```

### Editing a message
You can edit a message with [`editMessage`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-editMessage).

```Javascript!
skygearChat.editMessage(message, 'New Message.', null)
  .then(function (result) {
    console.log('Message edited!', result);
});
```

### Deleting a message
You can delete a message with [`deleteMessage`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-deleteMessage).

```Javascript!
skygearChat.deleteMessage(messageID, null)
  .then(function (result) {
    console.log('Message deleted!', result);
});
```

## Subscribing to new messages
### Subscribing to messages in a conversation

In order to get real time update of new messages, you can subscribe to a conversation with [`subscribe`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-subscribe)

```javascript!
//To subscribe all the conversations
skygearChat.subscribe(function handler(msgData) {
  // Handle Message data here
  if (msgData.record.conversation.id === currentId) {
      messages.push(msgData.record);
  }
});
```

### Subscribing to messages in all conversations 

Besides a specific conversation, you might want to get notified whenever there are new messages in any conversation you belong to. 

You can subscribe to all messages in your own user channel with  [`subscribe`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-subscribe)

```javascript!
//To subscribe all the conversations
skygearChat.subscribe(function handler(msgData) {
  // Handle Message data here
  messages.push(msgData.record);
  }
});
```

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

You can send the push notification to particular `userIds` and these can be retrieved by accessing the attribute `participantsIds` of `Conversation`.

Coming soon

### Receiving push notifications
Coming soon


## Receipt
Skygear Chat provides you with message receipt which includes status and timestamps information.

### Message status
You can make use of the following receipt status to indicate your message status.

- `Delivering` - The message is in the server but it is not yet delivered to the recipient device
- `Delivered` - The message is delivered to the recipient device but it is not yet read by the user
- `Read`- The message is delivered to the recipient device and it is ready by the user

`status` has 3 values.

| Status  | Description  |
| ------  | ------------ |
|`Delivering`|The message is being delivered, it is not yet received by the other party.|
|`Delivered `|The message is delivered, but it is not read yet.|
|`Read`|The message is delivered and read by some participants.|
|


### Subscribing to message status change
By subscribing to the status of a message, you can get the latest status of the message sent to other recipients with [`subscribe`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-subscribe).

### Marking messages as read
On the recipient client side, you need to update the message status if the message is read. 

```Java
skygearChat.markAsRead(messages, function handler(response) {
  console.log('Messages marked as read.');
});
```

Note: `messages` is a List of `Message`.

## Message unread count
### Conversation unread count
You can show the unread count of a `Conversation` with `unread_count`.

### Total unread count
You may wish to show the overall unread count of all conversations in the badge value of your app with [`getUnreadCount()`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-getUnreadCount).

```Javascript!
skygearChat.getUnreadCount().then(function (count) {
  console.log('Overall unread counts: ', count);
}, function (err) {
  console.log('Error: ', err);
});
```

### Resetting unread counts
The unread count can be reset by [marking the message as read](#marking-messages-as-read).

## Typing indicator

Skygear Chat provides real-time update to show the typing status of a user in a conversation.

There are 3 status:

- `begin` - User begins typing
- `pause` - User stopped typing
- `finished`- User stopped typing and the message is sent


### Subscribing to other's typing status

To get other's typing status in real time, you need to subscribe to it so that your app will listen for any typing status updates.

There are 2 APIs you can call:

- **[subscribeTypingIndicator](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-subscribeTypingIndicator)**: It's for subscribing to a specific conversation. (common use case)
- **[subscribeAllTypingIndicator](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-subscribeAllTypingIndicator)**: It's for subscribing to all typing indicator events.

```javascript!
skygearChat.subscribeTypingIndicator(conversation, function (payload) {
  indicateEl.textContent = JSON.stringify(payload);
});
```

Both callback functions return one variable as follows:

```javascript!
// All typing indicator
{
  "conversation/id1": {
    "user/id": {
      "event": "begin",
      "at": "20161116T78:44:00Z"
    },
    "user/id2": {
      "event": "begin",
      "at": "20161116T78:44:00Z"
    }
  }
}

// Typing indicator
{
  "user/id": {
    "event": "begin",
    "at": "20161116T78:44:00Z"
  },
  "user/id2": {
    "event": "begin",
    "at": "20161116T78:44:00Z"
  }
}
```

### Sending an User's typing status
To get typing status from other devices, you should always update your typing status to the server with [`sendTypingIndicator`](https://docs.skygear.io/js/chat/reference/latest/class/lib/container.js~SkygearChatContainer.html#instance-method-sendTypingIndicator).

```javascript!
messageInput.addEventListener("focus", function () {
  skygearChat.sendTypingIndicator(conversation, 'begin');
  // the state is sent to the server
});

messageInput.addEventListener("blur", function () {
  skygearChat.sendTypingIndicator(conversation, 'finished');
    // the state is sent to the server
});
```


## User online
Coming soon

## Best practices
### Caching message history locally
Skygear Chat does not cache message history in client device. You may consider to use the following libraries.

- [dirkbonhomme/js-cache](https://github.com/dirkbonhomme/js-cache)
- [pamelafox/lscache](https://github.com/pamelafox/lscache)
