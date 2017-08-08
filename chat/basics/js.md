---
title: Skygear Chat
---

[[toc]]

## Introduction

Skygear Chat is a collection of APIs to help you build real-time chat apps a lot easier.

Skygear Chat is built on top of the cloud database and the PubSub module. If you want to know more about how the underlying works, check out the guide for [cloud database](https://docs.skygear.io/guides/cloud-db/basics/js/) and [PubSub](https://docs.skygear.io/guides/pubsub/basics/js/).

Before we start, make sure you have already:

- switched on the Chat plug-in in the developer portal
- imported Chat SDK (skygear-chat) in your project

```javascript
const skygearChat = require('skygear-chat')
```
For more details, check out the [quick start](https://docs.skygear.io/guides/chat/quick-start/js/) for the chat module.

And here are a list of Sample Projects using Skygear JS Chat SDK:
* [Ionic Chat Demo](https://github.com/skygear-demo/ionic-chat-demo)
* [React Chat Demo](https://github.com/skygear-demo/react-chat-demo)

:::todo
* Shall include more details about the data type: Message, UserConversation and
  Conversation; although they are included as Example.
* We should also explain conversation options are configurable via
  [`updateConversation`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/container.js~SkygearChatContainer.html#instance-method-updateConversation)
* We should explain or include a link to the UIKit for quick chat application
  implementation
* We should mention the TypingIndicator Utility.
* Expand the Recipient status section
* Explain and include samples for User Online Indicator
:::

## Users
Most real-time chat apps require a user login system and you can do it with the Skygear user authentication module. It offers various sign up methods - sign up with email, sign up with username and sign up with Facebook and Google account.

Check out the guide for [user authentication](https://docs.skygear.io/guides/auth/basics/js/).

## Conversation basics

Conversation refers to the dialog between/amongst users. You can view it as a chatroom or a channel. You need to create a conversation before sending a message to a user.

Attributes of a conversation:

| Attributes  | Type             | Description  |
| ------ | ------------------- | ------------ |
| title  | <code>String</code> | |
| admin_ids | <code>JSON Array</code> | |
| participant_ids | <code>JSON Array</code> | |
| participant_count | <code>Number</code> | |
| distinct_by_participants | <code>Boolean</code> | |
| metadata | <code>JSON Object</code> | |
| last_message | <code>Reference</code> | |

### Creating a conversation

There are 2 APIs you can use to create a conversation:

- **[`createDirectConversation`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/container.js~SkygearChatContainer.html#instance-method-createDirectConversation)**: It's for one-to-one conversation
- **[`createConversation`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/container.js~SkygearChatContainer.html#instance-method-createConversation)**: It's for group conversation

Below is an example of creating a direct chat between the current user and Ben.

```javascript
skygearChat.createDirectConversation(userBen, 'Greeting')
  .then(function (conversation) {
    // a new direct conversation is created

    console.log('Conversation created!', conversation);
  }, function (err) {
    console.log('Failed to create conversation');
  });
```

Below is an example of creating a group conversation.

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
:::tips
**Fetching a user**

From the above example, you can see we that need a user object to create a conversation. To get a user, you can either use [`discoverUserByEmails`](https://docs.skygear.io/js/reference/v1/class/lib/auth.js~AuthContainer.html#instance-method-discoverUserByEmails) or [`discoverUserByUsernames`](https://docs.skygear.io/js/reference/v1/class/lib/auth.js~AuthContainer.html#instance-method-discoverUserByUsernames), depending on the credential you use for user authentication (i.e. [`signupWithEmail`](https://docs.skygear.io/js/reference/v1/class/lib/auth.js~AuthContainer.html#instance-method-discoverUserByEmails) or [`signupWithUsername`](https://docs.skygear.io/js/reference/v1/class/lib/auth.js~AuthContainer.html#instance-method-signupWithUsername)).

If you are using a 3rd party login provider, you need create write your own user identifier. Check out the user profile guide for more details.
:::

There are a few more attributes you can specify when you create a conversation:
`skygearChat.createConversation(partcipants, title, meta, options)`.

- **title**: That's the title of a conversation. It can be omitted.
- **meta**: It is for application specific purpose. Say in your app you have different type of conversations, you can specific it here.
- **options**: There are two options of a conversation, `distinctByParticipants` and `admins`. `distinctByParticipants` applies only to group conversation and `admins` applies to both conversation types. See more details more.

**DistinctByParticipants**

This option useful when you want to create group conversation with distinct participants.

If `DistinctByParticipants` is `True`, when you create a conversation with a group of people, and a conversation with the same group of people already exists, the call will return the existing conversation.

By default, `DistincyByParticipants` is `False`. That means every time you create a conversation with the same group of people, the call will return a new conversation.

```JavaScript!
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

// 5 mins later when the same group of people create a new conversation

skygearChat.createConversation(participants, conversationTitle , {} , {distinctByParticipants: true})
  .then(function (conversation) {
    // the app will return the same conversation created 5 mins ago

    console.log('Conversation created!', conversation);
  }, function (err) {
    console.log('Failed to create conversation');
  });

```
**Admins**

By default, all participants will be the admin of the conversation unless `admins` is specified in options.

In the below an example `userBen` will be the only admin of the conversation.

```JavaScript!
const participants = [userBen, userCharles, userDavid, userEllen];
const conversationTitle = "Greeting";

skygearChat.createConversation(participants, conversationTitle, {}, admins: [userBen]})
  .then(function (conversation) {
    // a new conservation is created
    // Ben will be the only admin of the conversation

    console.log('Conversation created!', conversation);
  }, function (err) {
    console.log('Failed to create conversation');
  });
```

The admin of a conversation has the following permissions:

- add or remove participants from a conversation
- add or remove admins from the conversation

### Getting all the conversations of a user has

In most chat apps, you need to display a list of conversations the currently logged in user has. Think about the list of conversations (main screen) of Whatsapp or Telegram.

To do so you can simply call [`getUserConversations()`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/container.js~SkygearChatContainer.html#instance-method-getUserConversations).

```javascript!
const includeLastMessage=True;
const page=1;
const pageSize=20;

skygearChat.getUserConversations(includeLastMessage, page, pageSize)
  .then(function (conversations) {

    /*
     * it returns all conversations the current user has
     * it returns the last message of every conversation
     * it displays the first page of the conversation list
     * each page has 20 conversations
     */

    console.log('All conversations', conversations);
  }, function (err) {
    console.log('Failed to retrieve conversations');
  })
```
In this call, you can specify 3 things:

- **includeLastmessage: boolean, optional, default=True** - Whether or not you want to get the last message of each of the conversation in the call.
- **page: number, optional, default=1** - Which page to display. Default to be the first page.
- **pageSize: number, optional, default=50** - The number of conversation shown per page.

`getUserConversations` will return a list of `UserConversation` objects, which
consist of information specific for each users:

| Attributes  | Type                | Description  |
| ------ | ------------------- | ------------ |
| unread_count  | Number | |
| last_read_message | Reference | |
| user | Reference | |
| conversation | Reference | |

There is a reference to the `Conversation` object, which consist of general
information about a Conversation, and consist of the following:

| Attributes  | Type                | Description  |
| ------ | ------------------- | ------------ |
| title  | String | |
| admin_ids | JSON Array | |
| participant_ids | JSON Array | |
| participant_count | Number | |
| distinct_by_participants | Boolean | |
| metadata | JSON Object | |
| last_message | Reference | |

### Leaving a conversation

You can call [`leaveConversation()`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/container.js~SkygearChatContainer.html#instance-method-leaveConversation) to leave a conversation.

```javascript!
skygearChat.leaveConversation(conversation)
  .then(function (conversation) {
    // you have left the conversation

    console.log('Left conversation', conversation);
  }, function (err) {
    console.log('Failed to leave the conversation');
  });
```

## Messages basics

Skygear Chat supports real time messaging. We will show how to send a message and get messages in real time below.

Message is the real content of a conversation. Skygear Chat supports 2 types of messages, one is plain text, the other one is assets.

Assets include files, images, voice message and video.

Attribute of messages:

| Attributes  | Type                | Description  |
| ------ | ------------------- | ------------ |
| body  | String | |
| conversation_status | String | Summary of receipt status |
| attachment | Asset | File |
| metadata | JSON Object | Any meta data for chat message |
| conversation_id | Reference | |

### Sending messages

To send a message, you can call [`createMessage()`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/container.js~SkygearChatContainer.html#instance-method-createMessage) and specify a target conversation.

This is an example of sending a red plain text message together with a file.

```javascript!
const message = "hello!";
const metadata = {
    textColor: 'red'
};
// Getting the asset file using jQuery
const assets = $('message-asset').files[0];


skygearChat.createMessage(conversation, message, metadata, assets)
  .then(function (result) {
    // a plain text message is sent
    // the message is sepecified to be red
    // a file is sent

    console.log('Message sent!', result);
});
```

### Getting messages in real time

To get messages in real time, you should subscribe to a conversation with the API [`subscribe`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/container.js~SkygearChatContainer.html#instance-method-subscribe). Once you have subscribed a conversation, your app will listen for new messages published from the server.

```javascript!
//To subscribe all the conversations
skygearChat.subscribe(function handler(msgData) {
  // Handle Message data here
  if (msgData.record.conversation_id.id === currentId) {
      messages.push(msgData.record);
  }
});
```

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
- update - when a message updated, or if the delivery or read status change (e.g. from `delivered` to `some_read` at `conversation_status`)
- delete - when a message was deleted

### Sending push notification for new messages

Use [Push Notification API](https://docs.skygear.io/guides/push-notifications/basics/js/) to implement any push notification, needed.

## Chat history
When users enter a conversation, you may need to display the chat history of the conversation.

You can call [`getMessages()`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/container.js~SkygearChatContainer.html#instance-method-getMessages) to get an array of message of a conversation.

```JavaScript
const limit=10;
const beforeTime = new Date();

skygearChat.getMessages(conversation, limit, beforeTime)
  .then(function (messages) {
    // messages contains the latest 10 messages
  });
```
Messages are queried by `limit` and `beforeTime` in this call:

1. **limit: number, optional, default=50** - It's the number of messages the call will return. If the number is too large, it may result in timeout.
2. **beforeTime: date** - It specify from which time the query should get the message.

## Message receipt status

Skygear Chat provides real-time update to show the receipt status of a message.

There are 3 status:

- `Delivering` - The message is in the server but it is not yet delivered to the recipient device
- `Delivered` - The message is delivered to the recipient device but it is not yet read by the user
- `Read`- The message is delivered to the recipient device and it is ready by the user

### Subscribing to message status change
By subscribing to the status of a message, you can get the latest status of the message sent to other recipients with [`subscribe`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/index.js~SkygearChatContainer.html#instance-method-subscribe).

### Marking messages as read
On the recipient device, you need to update the message status if the message is read with [`markAsRead`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/index.js~SkygearChatContainer.html#instance-method-markAsDelivered)

## Typing indicator

Skygear Chat provides real-time update to show the typing status of a user in a conversation.

There are 3 status:

- `begin` - User begins typing
- `pause` - User stopped typing
- `finished`- User stopped typing and the message is sent

### Sending my typing status

First of all, you need to send the users' typing status to the server using [`sendTypingIndicaton`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/container.js~SkygearChatContainer.html#instance-method-sendTypingIndicator).

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
### Subscribing to other's typing status

To get other's typing status in real time, you need to subscribe to it so that your app will listen for any typing status updates.

There are 2 APIs you can call:
- **[subscribeTypingIndicator](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/container.js~SkygearChatContainer.html#instance-method-subscribeTypingIndicator)**: It's for subscribing to a specific conversation. (common use case)
- **[subscribeAllTypingIndicator](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/container.js~SkygearChatContainer.html#instance-method-subscribeAllTypingIndicator)**: It's for subscribing to all typing indicator events.

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

## Unread counts

There are 2 APIs you can use to get unread counts.
- **[`getUnreadMessageCount`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/container.js~SkygearChatContainer.html#instance-method-getUnreadMessageCount)**: It's to get the unread count of a specific conversation
- **[`getUnreadCount`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/container.js~SkygearChatContainer.html#instance-method-getUnreadCount)**: It's to get the overall unread counts of all the conversations of a user

```javascript
skygearChat.getUnreadMessageCount(conversationA).then(function (count) {
  console.log('Unread counts of conversation A: ', count);
}, function (err) {
  console.log('Error: ', err);
});

skygearChat.getUnreadCount().then(function (count) {
  console.log('Overall unread counts: ', count);
}, function (err) {
  console.log('Error: ', err);
});
```
### Resetting unread counts
The unread count can be reset by [marking the message as read](#marking-messages-as-read).

## Push notification

Use [Push Notification API](https://docs.skygear.io/guides/push-notifications/basics/js/) to implement any push notification, needed.

## User Online Indicator
Coming soon

## Best practices
### Caching message history locally
Skygear Chat does not have support on caching message history locally. However you can always achieve offline caching with other tools.

There are some good libraries helping you to cache messages offline:
- [dirkbonhomme/js-cache](https://github.com/dirkbonhomme/js-cache)
- [pamelafox/lscache](https://github.com/pamelafox/lscache)
