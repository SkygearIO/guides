---
title: Skygear Chat
---
Skygear Chat is a collection of APIs to help you build Chat apps much easier.

## Enabling on Chat Plugin on Skygear Portal 


To set up a JavaScript project using Skygear, please refer to the [Quick Start](https://docs.skygear.io/guides/get-started/js/).

You also need to import `skygear-chat` inside your project:

```JavaScript
const skygearChat = require('skygear-chat');
```

## Creating conversations

In order to send messages, you need to create a conversation first. You can consider **conversations** as chatrooms or channels in your application.

There are two types of conversations in Skygear:
- **Direct Conversation**, support chatting between one user to another
- **Group Conversation**, support chatting among 2 or more users

### Creating direct conversations 
You can use `createDirectConversation(user: User, title: string, meta: object, options: object)` to create a conversation with another user. Please specify the user ID as `user`. 
  
This example shows how to create a direct chat between the current user and `userBen`.

You can also set up optional `metadata` in your conversation for custom configurations.

```JavaScript
skygearChat.createDirectConversation(userBen, 'Greeting')
  .then(function (conversation) {
    console.log('Conversation created!', conversation);
  }, function (err) {
    console.log('Failed to create conversation'');
  });
```

### Creating Group Conversations

Besides direct chats, you can also create a group conversation with 2 or more people. 

Instead of a single `user` as the parameter, `createConversation` accepts a list of `participants`. A new conversation will be created with the IDs given as participants.

Example of using ` createConversation(participants: []User, title: string, meta: object, options: object)`

```JavaScript
skygearChat.createConversation([userBen, userCharles, userDavid, userEllen], 'Greeting')
  .then(function (conversation) {
    console.log('Conversation created!', conversation);
  }, function (err) {
    console.log('Failed to create conversation');
  });
```

You may also specify metadata with the `meta` parameter when creating a conversation.

### Creating group chat by distinct participants (distinctByParticipants)

By default, if you try to create conversations with same list of participants with `createConversation`. You will eventually create different conversations with the identical participants.

This may or may be a desire behavior in your application depends on your app design.

If you want to make sure each conversation contains distinct participants, you can set `distinctByParticipants` in options to be `true`.

By default, `distinctByParticipants` is `false`.

```JavaScript
skygearChat.createConversation([userBen, userCharles, userDavid, userEllen], 'Greeting' , {} , {distinctByParticipants: true})
  .then(function (conversation) {
    console.log('Conversation created!', conversation);
  }, function (err) {
    console.log('Failed to create conversation');
  });
```

Upon calling the above function twice, you will receive an error as duplicated conversations.

### Setting admins

All participant will be admin unless specific in `options.admins`

```JavaScript
skygearChat.createConversation([userBen, userCharles, userDavid, userEllen], 'Greeting' , {} , {admins: [userBen]})
  .then(function (conversation) {
    console.log('Conversation created!', conversation);
  }, function (err) {
    console.log('Failed to create conversation');
  });
```

In this conversation, `userBen` will be the only admin.

### Fetching existing conversations

You can also get all conversations of the current user.


```javascript
skygearChat.getConversations()
  .then(function (conversations) {
    console.log('All conversations', conversations);
  }, function (err) {
    console.log('Failed to retrieve conversations');
  })
```

Or a specific conversation given its `id`.

```javascript
skygearChat.getConversation(conversationId)
  .then(function (conversation) {
    console.log('Retrieved conversation', conversation);
  }, function (err) {
    console.log('Cannot find conversation with id', conversationId);
  })
```

### Leaving conversations 

To leave a conversation, you can call `leaveConversation(conversation: string)`.

```JavaScript
skygearChat.leaveConversation(conversationID)
  .then(function (conversation) {
    console.log('Left conversation', conversation);
  }, function (err) {
    console.log('Failed to leave the conversation');
  });
```

## Managing conversation participants
At some point of your conversation, you may wish to update the participant list. You may add or remove participants in a conversation.

### Adding users to conversation 

You can add users to an existing conversation with `addParticipants(conversation: string, participants: []User)`

```javascript
skygearChat.addParticipants(conversationId, [userBen])
  .then(function () {
    console.log('Successfully added', userBen);
  }, function (err) {
    console.log('Failed to add participant', userBen);
  })
```

### Removing users from conversation 


To remove users from a conversation, you can call `removeParticipants(conversation: string, participants: []User)`

```javascript
skygearChat.removeParticipants(conversationId, [userBen])
  .then(function () {
    console.log('Successfully removed', [userBen]);
  }, function (err) {
    console.log('Failed to remove', userBen)
  });
```


### Admin

An admin of the conversation has the following permissions:
1. add or remove participants from to conversation, 
2. add or remove admins from the conversation,
3. delete the conversation

The number of admins in a conversation is unlimited, so you may add everyone as an admin.

#### Adding admins

You can call [`addAdmins(conversation: string, admins: []User)`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/index.js~SkygearChatContainer.html#instance-method-addAdmins) to add a list of users as admins in a conversation.


#### Removing admins

To remove an existing admin from a conversation, you can call [`removeAdmins(conversation: string, admins: []User)`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/index.js~SkygearChatContainer.html#instance-method-removeAdmins).

## Chat history
### Loading all messages from a conversation 


When users get into the chatroom, you may want to load the messages of the conversation with `getMessages(conversation: Conversation, limit: number, beforeTime: Date)`. You can specify the limit of the messages in `limit` and the time constraint for the message in `beforeTime`.

```JavaScript
const currentTime = new Date();

skygearChat.getMessages(conversation, 10, currentTime)
  .then(function (messages) {
    // messages contains the latest 10 messages
  });
```

## Sending messages
To send a message, just call `createMessage` and specify a target conversation. You can also set metadata and attachments accordingly. 

### Plain text

` createMessage(conversation: Conversation, body: string, metadata: object, asset: File)`


To send a text message, just set `body` in `createMessage`.

```
skygearChat.createMessage(
  conversation,
  'Hello!'
).then(function (result) {
  console.log('Message sent!', result);
});
```

### Metadata
Besides the body of the message, you may wish to specify metadata in your message. For example, special format or color of your message.

`metadata` can contain a JSON format object
```
skygearChat.createMessage(
  conversation,
  'Hello!',
  {textColor: 'red'}
).then(function (result) {
  console.log('Message sent!', result);
});
```
### Files
If you would like to send files via Skygear Chat, you can upload a file as an asset.

```
skygearChat.createMessage(
  conversation,
  'Hello! See the attachment!',
  {textColor: 'red'},
  $('message-asset').files[0]
).then(function (result) {
  console.log('Message sent!', result);
});
```

## Subscribing to new messages
### Subscribing to messages in a conversation

In order to get real time update of new messages, you can subscribe to a conversation with `subscribe`



### Subscribing to messages in all conversations

Besides a specific conversation, you might want to get notified whenever there are new messages in any conversation you belong to. 

You can subscribe to all messages in your own user channel with `subscribe()`

```
// Set subscribe handler to messages
skygearChat.subscribe(function handler(data) {
  // Handle Message data here
  appendMessageToConversationList(data);
});
```

## Displaying unread count
### Conversation unread count
You can show the unread count for different conversations in the conversation list.

```
skygearChat.getUnreadMessageCount(conversation).then(function (count) {
  console.log('Total unread count: ', count);
}, function (err) {
  console.log('Error: ', err);
});
```

### Overall unread count
You may wish to show the overall unread count of all conversations in the badge value of your app.

```
skygearChat.getUnreadCount().then(function (count) {
  console.log('Total unread count: ', count);
}, function (err) {
  console.log('Error: ', err);
});
```

### Resetting the unread count
The unread count can be reset by [marking the messages as read](#marking-messages-as-read).

## Typing indicator
The typing indicator has these three states:

- `begin` - User began typing
- `pause` - User stopped typing
- `finished`- User stopped typing and the message is sent

You can make good use of these states to implement the typing indicator feature in your app.

### Subscribing to typing indicator
Skygear Chat provides real-time update to typing indicators in a particular conversation.

```
skygearChat.subscribeTypingIndicator(conversation, function (payload) {
  indicateEl.textContent = JSON.stringify(payload);
});
```

You can also subscribe to all typing indicators in all conversations the current user is involved with [`subscribeAllTypingIndicator()`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/index.js~SkygearChatContainer.html#instance-method-subscribeAllTypingIndicator)

### Sending my typing status
To get typing status from other devices, you should always update your typing status to the server with `sendTypingIndicaton(conversation, state)`

```
messageInput.addEventListener("focus", function () {
  skygearChat.sendTypingIndicator(conversation, 'begin');
});

messageInput.addEventListener("blur", function () {
  skygearChat.sendTypingIndicator(conversation, 'finished');
});
```
## Recipient status
Skygear Chat is helpful for displaying recipient status such as *"Delivering"*, *"Delivered"* or *"Seen"*. 

### Message status
You can make use of the following receipt status to indicate your message status.

### Subscribing to message status change
By subscribing to the status of a message, you can get the latest status of the message sent to other recipients with [`subscribe(handler: function)`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/index.js~SkygearChatContainer.html#instance-method-subscribe).

### Marking messages as read
On the recipient client side, you need to update the message status if the message is read with [`markAsDelivered(messages)`](https://doc.esdoc.org/github.com/skygeario/chat-SDK-JS/class/lib/index.js~SkygearChatContainer.html#instance-method-markAsDelivered)


## Push notification
### Sending push notifications to conversation participants

We can send the push notification to particular `userIds` and these can be retrieved by accessing the attribute `participantsIds`.

Coming Soon

### Receiving push notifications
Coming soon

## User Online Indicator
### User online indicator status
Coming soon
### Subscribing to user online indicator
Coming soon


## Best practices
### Caching message history locally
Skygear Chat does not have support on caching message history locally. However you can always achieve offline caching with other tools.

There are some good libraries helping you to cache messages offline:
- [dirkbonhomme/js-cache](https://github.com/dirkbonhomme/js-cache)
- [pamelafox/lscache](https://github.com/pamelafox/lscache)
