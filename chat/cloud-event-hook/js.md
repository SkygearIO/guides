---
title: Skygear Chat Cloud Function & Event Hooks
---

## Overview
Skygear Chat event hooks are executed after any of the following actions are executed.

- Message
    - Sending a message
    - Editing a message
    - Deleting a message
    - Typing a message
- Conversation
    - Creating a conversation
    - Updating a conversation
    - Deleting a conversation
    - Adding participants to a conversation
    - Removing participants from a conversation

The below code snippet prints a message to server log after an user send a message.

```javascript
const skygearChatCloud = require("skygear-chat/cloud");
const skygearCloud = require("skygear/cloud");

skygearChatCloud.afterMessageSent((message, conversation, participants, context) => {
  console.log("Message [%s] is sent to conversation [%s].", message.body, conversation.title);
});
```

Note: Currently only one of the same kind hooks can be registered.

## Skygear Cloud Function
Please refer to [Introduction to Cloud Functions](https://docs.skygear.io/guides/cloud-function/intro-and-deployment/) for the basics of cloud function.

## Installing `skygear-chat` in cloud
Skygear Chat SDK JS is not included in cloud function by default. Therefore, you need to specify the dependency in `package.json`

```json
{
    "name": "your-project",
    "version": "1.0.0",
    "description": "Describe your project here.",
    "main": "index.js",
    "dependencies": {
        "skygear-chat": "^1.3.4",
        "skygear": "^1.3.0"
    }
}
```

## Available Event Hooks
Event hooks are implemented in [Skygear Chat SDK JS](https://github.com/skygeario/chat-SDK-JS) project. Therefore, you need to import `skygear-chat/cloud` in order to use event hooks in cloud function.

The followings are the available event hooks.

- `afterMessageSent`: to run after a message is sent
- `afterMessageUpdated`: to run after a message is updated
- `afterMessageDeleted`: to run after a message is deleted
- `typingStarted`: to run after typing is started
- `afterConversationCreated`: to run after a conversation is created
- `afterConversationUpdated`: to run after a conversation is update
- `afterConversationDeleted`: to run after a conversation deleted
- `afterUsersAddedToConversation`: to run after one or more users are added to a conversation
- `afterUsersRemovedFromConversation`: to run after one or more users are removed from the conversation

All decorated functions are not required to return any value.


### Decorated Function Interfaces and Parameters

Depending on decorators, decorated functions contain any of the following parameters.

| Tables        | Description                 |
| ------------- |:---------------------------:|
| `message`     | Message Record              |
| `conversation`| Conversation Record         |
| `participants`| Array of User Record        |
| `events`      | Array of Typing Events      |
| `newUsers`    | Array of User Records       |
| `oldUsers`    | Array of User Records       |
| `context`     | Object containing `userId`  |


#### Function Interfaces
Below are the list of function interfaces of each decorator.

##### `afterMessageSent`
```javascript
skygearChatCloud.afterMessageSent(
  (message, conversation, participants, context) => {
  }
);
```
##### `afterMessageUpdated`
```javascript
skygearChatCloud.afterMessageUpdated(
  (message, conversation, participants, context) => {
  }
);
```
##### `afterMessageDeleted`
```javascript
skygearChatCloud.afterMessageDeleted(
  (message, conversation, participants, context) => {
  }
);
```
##### `typingStarted`
```javascript
skygearChatCloud.typingStarted(
  (conversation, participants, events, context) => {
  }
);
```
##### `afterConversationCreated`
```javascript
skygearChatCloud.afterConversationCreated(
  (conversation, participants, context) => {
  }
);
```
##### `afterConversationUpdated`
```javascript
skygearChatCloud.afterConversationUpdated(
  (conversation, participants, context) => {
  }
);
```
##### `afterConversationDeleted`
```javascript
skygearChatCloud.afterConversationDeleted(
  (conversation, participants, context) => {
  }
);
```
##### `afterUsersAddedToConversation`
```javascript
skygearChatCloud.afterUsersAddedToConversation(
  (conversation, participants, newUsers, context) => {
  }
);
```
##### `afterUsersRemovedFromConversation`
```javascript
skygearChatCloud.afterUsersRemovedFromConversation(
  (conversation, participants, oldUsers, context) => {
  }
);
```


## Real Life Example: Push Notification
The following example demonstrates push notification implementation with Skygear API and Skygear Chat event hook. Function `after_message_sent_hook` is called after a message is sent. `otherUserIds` and `notification` are evaluated from `participants` and `message` respectively. Lastly, `sendToUser ` is called and participants' devices receive notifications.

```javascript
const skygearChatCloud = require('skygear-chat/cloud');
const skygearCloud = require('skygear/cloud');
const container = new skygearCloud.CloudCodeContainer();

container.apiKey = skygearCloud.settings.apiKey;
container.endPoint = skygearCloud.settings.skygearEndpoint + '/';

skygearChatCloud.afterMessageSent((message, conversation, participants, context) => {
  const title = conversation.title;
  const otherUserIds = participants.filter(p => p._id && p._id != context.userId).map(p => p._id);
  const currentUser = participants.find(p => p._id == context.userId);
  let body = '';
  if (message.body) {
    body = currentUser.username + ": " + message.body;
  } else {
    let fileMessage = "a file";
    if (message.attachment) {
      if (message.attachment.contentType.startsWith("audio/")) {
        fileMessage = "an audio file";
      }
      if (message.attachment.contentType.startsWith("image/")) {
        fileMessage = "an image file"
      }
    }
    body = currentUser.username + " sent you " + fileMessage + ".";
  }

  const apnsPayload = {
    'aps': {
      'alert':
        {'title':title,
         'body': body}
      },
    'from': 'skygear',
    'operation': 'notification'
  };
  const gcmPayload = {
    'notification': {
      'title': title,
      'body': body
    }
  }
  const payload = {'gcm': gcmPayload, 'apns': apnsPayload};
  container.push.sendToUser(otherUserIds, payload);
});

```


## See Also
[Demo project](https://github.com/skygear-demo/cloud-chat-demo-js)
