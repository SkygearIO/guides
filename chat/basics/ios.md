---
title: Skygear Chat
---

[[toc]]

# Skygear Chat Basic (iOS)
Skygear Chat is a collection of APIs to help you build Chat apps much easier.

Skygear Chat is built on top of the cloud database and the PubSub module. If you want to know more about how the underlying works, check out the guide for [cloud database](https://docs.skygear.io/guides/cloud-db/basics/ios/) and [PubSub](https://docs.skygear.io/guides/pubsub/basics/ios/).

## Enabling Chat
To start using Skygear Chat feature, make sure you have already enabled Chat in the Plug-ins tab in your [Skygear Portal] (https://portal.skygear.io). Please refer to [Quick Start](https://docs.skygear.io/guides/chat/quick-start/ios/) for the instructions.

## Users
Most real-time chat apps require a user login system and you can do it with the Skygear user authentication module. It offers various sign up methods - sign up with email, sign up with username and sign up with Facebook and Google account.

Check out the guide for [User Authentication Basics](https://docs.skygear.io/guides/auth/basics/ios/) and [User Profile Best Practices](https://docs.skygear.io/guides/auth/user-profile/ios/).

## Conversations

In order to send messages, you need to create a conversation first. You can consider conversations as chatrooms or channels in your application.

There are two types of conversations in Skygear Chat:

- **Direct Conversation**, a chatting group between one and another user
- **Group Conversation**,  a chatting group among 2 or more users

### Creating direct conversations

You can use [`createDirectConversation`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)createDirectConversationWithUserID:title:metadata:completion:) to create a conversation with another user. Please specify the user ID as `userID`.

```swift
SKYContainer.default().chatExtension?.createDirectConversation(userID: userBen,
    title: "Chat with Ben",
    metadata: nil,
    completion: { (conversation, error) in
        if error != nil {
            print ("Create direct conversation failed." +
                   "Error:\(error.localizedDescription)")
            return
        }
        print ("Created direct conversation")
})
```

The above example creates a direct chat between the current user and `userBen`.

You can also set up an optional parameter, `metadata`, for customization.

### Creating group conversations

Besides direct chats, you can also create a group conversation with 2 or more people.

Instead of a single `userID` parameter, [`createConversation`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)createConversationWithParticipantIDs:title:metadata:completion:) accepts a list of users called `participantIDs`. A new conversation will be created with the given participant IDs.

```swift
SKYContainer.default().chatExtension?.createConversation(
    participantIDs: [userBen, userCharles, userDavid, userEllen],
    title: "Random chat group",
    metadata: nil,
    completion: { (conversation, error) in
        if error != nil {
            print ("Create conversation failed." +
                   "Error:\(error.localizedDescription)")
            return
        }
        print ("Created Conversation")
})
```

### Creating group chat by distinct participants (`distinctByParticipants`)

By default, [`createConversation`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)createConversationWithParticipantIDs:title:metadata:completion:) allows you to create multiple conversations with identical participants. However, depending on application design, it may not be a desired behavior. Therefore, Skygear Chat provides `distinctByParticipants` option, which enables distinctive participants in every conversation.

The default value of `distinctByParticipants` is `false`. If it is set to be `true`, then the participants cannot be identical to any existing conversation. Otherwise, an error will be returned.

```swift
SKYContainer.default().chatExtension?.createConversation(
    participantIDs: [userBen, userCharles, userDavid],
    title: "Our chat group",
    metadata: nil,
    adminIDs: nil,
    distinctByParticipants: true,
    completion: { (conversation, error) in
        if let err = error as NSError? {
             if let conversationId = err.userInfo["conversation_id"] as? String {
                 print("Conversation already exists " +
                   conversationId)
             } else {
                 print ("Create conversation failed. " +
                   "Error:\(error.localizedDescription)")
             }
        }
        print ("Created conversation")
})
```

If the conversation already exists, then `completion` callback returns an error object. You can retrieve the original conversation ID via `err.userInfo["conversation_id"]`.

Note: `distinctByParticipants` will be `false` automatically after participant list is being altered.

There are a few more attributes you can specify when you create a conversation with [`createConversation`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)createConversationWithParticipantIDs:title:metadata:completion:).

- **title**: That's the title of a conversation. It can be omitted.
- **meta**: It is for application specific purpose. Say in your app you have different type of conversations, you can specific it here.

### Setting admins
All users in `participantIDs` will be administrators unless `adminIDs` is specified in [`createConversation`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)createConversationWithParticipantIDs:title:metadata:completion:).


```swift
SKYContainer.default().chatExtension?.createConversation(
   participantIDs: [userBen, userCharles, userDavid],
   title: "Ben's world",
   adminIDs: [userBen],
   distinctByParticipants: false,
   completion: { (conversation, error) in
      if error != nil {
          print("Unable to admin. " +
                "Error:\(error.localizedDescription)")
          return
      }

      print("Added as admin")
})
```

In the above example, `userBen` will be the only administrator.


### Fetching existing conversations
In [`SKYKitChat`](https://docs.skygear.io/ios/chat/reference/latest/), `SKYConversation` represents a conversation, it contains information of a conversation that is shared among all participants, such as `participantIds` and `adminIds`.

There are two methods to fetch existing conversations:

- [`fetchConversationsWithCompletion:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)fetchConversationsWithCompletion:)
- [`fetchConversationWithConversationID:fetchLastMessage:completion:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)fetchConversationWithConversationID:fetchLastMessage:completion:)

Fetch all conversations which involves the current user. The parameter `fetchLastMessage` can be used for fetching the last message and last read message in the conversation.

```swift
SKYContainer.default().chatExtension?.fetchConversations(
    fetchLastMessage: false,
    completion: { (conversations, error) in
    if let err = error {
        print ("Error when fetching conversations. " +
               "Error:\(err.localizedDescription)")
        return
    }

    if let fetchedConversations = conversations {
        print("Fetched \(fetchedConversations.count) conversations.")
    }
})
```

Please refer to the documentation for more options [here](http://cocoadocs.org/docsets/SKYKitChat/1.1.0/Classes/SKYChatExtension.html).




### Leaving conversations
To leave a conversation, you can call [`leave(conversationID:completion:)`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)leaveConversation:completion:) on the chat extension.


```swift
SKYContainer.default().chatExtension?.leave(
    conversationID: conversationID!) { (error) in
        if error != nil {
            print("Unable to Leave Conversation. " +
                  "Error:\(error.localizedDescription)")
            return
    }

    print("Left Conversation")
)
```

## Managing conversation participants
At some point of your conversation, you may wish to update the participant list. You may add or remove participants in a conversation.

### Adding users to conversation

You can add users to an existing conversation with [`addParticipantsWithUserIDs:toConversation:completion:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)addParticipantsWithUserIDs:toConversation:completion:). In the following code, `userBen`, `userCharles`, `userDavid` and `userEllen` will be added to `conversation`.

```swift
SKYContainer.default().chatExtension?.addParticipants(
    userIDs: [userBen, userCharles, userDavid, userEllen],
    to: conversation,
    completion: { (conversation, error) in
        if error != nil {
            print ("Unable to add the users. " +
                   "Error:\(error.localizedDescription)")
            return
        }

        print ("Users added to the conversation.")
})
```

### Removing users from conversation

To remove users from a conversation, you can call [`removeParticipantsWithUserIDs:fromConversation:completion:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)removeParticipantsWithUserIDs:fromConversation:completion:). In the following code, `userBen` and `userCharles` will be removed from `conversation`.

```swift
SKYContainer.default().chatExtension?.removeParticipants(
    userIDs: [userBen, userCharles],
    from: conversation,
    completion: { (conversation, error) in
        if error != nil {
            print ("Unable to remove the users. " +
                   "Error:\(error.localizedDescription)")
            return
        }

        print ("Users removed from the conversation")
})
```


### Admin

An admin of the conversation has the following permissions:

1. add or remove participants from to conversation,
2. add or remove admins from the conversation and
3. delete the conversation

The number of admins in a conversation is unlimited, so you may add everyone as an admin.

#### Adding admins

You can add admins to a conversation via calling [`addAdminsWithUserIDs:toConversation:completion:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)addAdminsWithUserIDs:toConversation:completion:). In the following example, `userDavid` and `userEllen` are added as admins of the conversation.

```swift
SKYContainer.default().chatExtension?.addAdmins(
    userIDs: [userDavid, userEllen],
    to: conversation,
    completion: { (conversation, error) in
        if error != nil {
            print("Unable to add the admins. " +
                   "Error:\(error.localizedDescription)")
            return
    }

    print("Admins added")
})
```
#### Removing admins
To remove admins from a conversation, use [`removeAdminsWithUserIDs:fromConversation:completion:`](removeAdminsWithUserIDs:fromConversation:completion:). In the following example, `userDavid` and `userEllen` are no longer admins.

```swift
SKYContainer.default().chatExtension?.removeAdmins(
    userIDs: [userDavid, userEllen],
    from: conversation,
    completion: { (conversation, error) in
        if error != nil {
            print("Unable to remove the admins. " +
                   "Error:\(error.localizedDescription)")
            return
    }

    print("Admins removed")
})
```
## Messages
Skygear Chat supports real time messaging. A message is the real content of a conversation. Skygear Chat supports 2 types of messages, one is plain text, the other one is assets. Assets include files, images, voice message and video.

### Loading messages from a conversation

When users get into the chatroom, you may call [`fetchMessagesWithConversation:limit:beforeTime:order:completion:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)fetchMessagesWithConversation:limit:beforeTime:order:completion:) to load the messages of the conversation. You can specify the limit of the messages in `limit` and the time constraint for the message in `beforeTime`.

The completion function would get called twice, once from cache and once from server. There is a boolean flag `isCached` in the completion function parameter reflecting if the messages are fetched from local cache or server.

```swift
SKYContainer.default().chatExtension?.fetchMessages(
    conversation: conversation,
    limit: 100,
    beforeTime: nil,
    order: nil,
    completion: { (messages, isCached, error) in
        if error != nil {
            print ("Messages cannot be fetched. " +
                   "Error:\(error.localizedDescription)")
        }

        if isCached {
            print ("Messages fetched from cache")
        }

        print ("Messages fetched")
})
```

### Sending messages

A message in Skygear Chat is a `SKYMessage` record.

To send a text message, just create a `SKYMessage` and specify the `body` of your message. [`addMessage:toConversation:completion:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)addMessage:toConversation:completion:).


```swift
let message = SKYMessage()
message.body = "Hello!"

SKYContainer.default().chatExtension?.addMessage(message,
    to: conversation) { (message, error) in
        if let err = error {
            // The message cannot be added, you should handle
            // this error.
            print("Send message error: \(err.localizedDescription)")
            return
        }

        // The message is added to server.
        // This is a good time to update your messages view with the added
        // message.
        updateMessagesView(message)
}
```

When adding a message to a conversation, the operation is created and is added
to the local cache store. The completion handler is called
when the message is added to server.

#### Handling failed messages

If messages failed to save, you can obtain them from
`fetchOutstandingMessageOperations(conversationID:operationType:completion:)`.

Outstanding message operations contain messages that are not yet synchronized
to the server. You can fetch outstanding message operations to display to
the user which messages are failed to synchronize, so that you can present
user with options whether to retry or cancel the operation.

Here is an example:

```swift
SKYContainer.default().chatExtension?.fetchOutstandingMessageOperations(
    conversationID: self.conversation!.recordID().recordName,
    operationType: SKYMessageOperationType.add
    completion: { (operations) in
        // Handle operation here. A typical example is to show
        // these messages alongside sent ones.
        for operation in operations {
            addFailedMessagesToView(operation.message)
        }
    })
```

To retry a message operation, calls `retry(messageOperation:completion:)`:

```swift
SKYContainer.default().chatExtension?.fetchOutstandingMessageOperations(
    messageID: message.recordID().recordName,
    operationType: SKYMessageOperationType.add
    completion: { (operations) in
        guard let operation = operations.first else {
            return
        }

        SKYContainer.default().chatExtension?.retry(messageOperation: operation, completion: nil)
    })
```

Calls `cancel(messageOperation:)` to cancel the message operation instead.

#### Plain Text

To send a text message, just create a `SKYMessage` and specify the `body` of your message.

Then you can use the [`addMessage:toConversation:completion:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)addMessage:toConversation:completion:) send a conversation (it works for both direct conversations or group conversations).

```swift
let message = SKYMessage()
message.body = "Hello!"

SKYContainer.default().chatExtension?.addMessage(message,
    to: conversation) { (message, error) in
        if let err = error {
            print("Send message error: \(err.localizedDescription)")
            return
        }

        if message != nil {
            print("Send message successful")
        }
}
```

#### Metadata

Besides the body of the message, you may wish to specify metadata in your message. For example, special format or color of your message.

`metadata` can contain a JSON format object

```swift
let message = SKYMessage()
    message.body = "Hello! See this photo!"
    message.metadata = {"text-color":"#ff0000"}

// Then send the message
```
#### Files

If you would like to send files via Skygear Chat, you can upload a file as [`SKYAsset`](https://docs.skygear.io/ios/reference/latest/Classes/SKYAsset.html).

```swift
guard let asset = SKYAsset(data: imageData) else {
    return
}

SKYContainer.default().uploadAsset(asset) { (uploadedAsset, error) in
    if error != nil {
        print ("Upload Asset failed. " +
               "Error:\(error.localizedDescription)")
        return
    }
    // Send the message after uploading the asset
}
```

### Editing a message
You can edit a message with [`editMessage:withBody:completion:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)editMessage:withBody:completion:).

```swift
SKYContainer.default().editMessage(message, with: newMessageBody, completion: { (result, error) in
	if let err = error {
	    print(err.localizedDescription)
	    return
	}

    print("Edit message successful")
})
```

### Deleting a message
You can delete a message with [`deleteMessage:inConversation:completion:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)deleteMessage:inConversation:completion:).

```swift
SKYContainer.default().chatExtension?.deleteMessage(message, in: conversation) { (conversation, error) in
    if let err = error {
        print(err.localizedDescription)
        return
    }
    print("Message Deleted.")
}
```

## Subscribing to new messages
### Subscribing to messages in a conversation

In order to get real time update of new messages, you can subscribe to a conversation with [`subscribeToMessagesInConversation:handler:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)subscribeToMessagesInConversation:handler:)

```swift
SKYContainer.default().chatExtension?.subscribeToMessages(
    in: conversation,
    handler: { (event, message) in
        print("Received message event")
})
```

### Subscribing to messages in all conversations

Besides a specific conversation, you might want to get notified whenever there are new messages in any conversation you belong to.

You can subscribe to all messages in your own user channel with  [`subscribeToUserChannelWithCompletion:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)subscribeToUserChannelWithCompletion:)

```swift
SKYContainer.default().chatExtension?.subscribeToUserChannelWithCompletion(
    completion: { (error) in
        print("Received message event")
})
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

You can send the push notification to particular `userIds` and these can be retrieved by accessing the attribute `participantsIds` of `SKYConversation`.

```swift
let apsInfo = SKYAPSNotificationInfo()
apsInfo.alertBody = text

let info = SKYNotificationInfo()
info.apsNotificationInfo = apsInfo

let operation = SKYSendPushNotificationOperation(
    notificationInfo: info,
    userIDsToSend: conversation.participantIds)
operation?.sendCompletionHandler = { (userIds, error) in
    if error != nil {
        print("Error on sending push notification. " +
              "Error: \(error?.localizedDescription)")
        return
    }

    if let userIds = userIds {
        print ("Sent notifications to \(userIds.count)")
    }
}

SKYContainer.default().add(operation)
```

### Receiving push notifications
Implement the handling of push notification in `AppDelegate`:
```swift
func application(_ application: UIApplication, didReceiveRemoteNotification
    userInfo: [AnyHashable : Any],
    fetchCompletionHandler completionHandler:
        @escaping (UIBackgroundFetchResult) -> Void) {
    let aps = userInfo["aps"] as! [String: AnyObject]
    print ("Received push notification: \(aps["alert"]?["body"])")
}
```

## Receipt
Skygear Chat provides you with message receipt which includes status and timestamps information.

### Message status
You can make use of the following receipt status to indicate your message status.

`SKYMessageConversationStatus` has 4 values.

| Status  | Description  |
| ------  | ------------ |
|`SKYChatReceiptStatusDelivering`|The message is being delivered, it is not yet received by the other party.|
|`SKYChatReceiptStatusDelivered`|The message is delivered, but it is not read yet.|
|`SKYChatReceiptStatusRead`|The message is delivered and read by some participants.|
|`SKYMessageConversationStatusAllRead`|The message is delivered and read by all participants.|


### Subscribing to message status change

By subscribing to [`SKYChatReceiptStatus`](https://docs.skygear.io/ios/chat/reference/latest/Enums/SKYChatReceiptStatus.html) in a message, you can get the latest status of the message sent to other recipients.

```swift
switch (message.conversationStatus) {
    case .allRead:
        print ("Message read by all.")
    case .someRead:
        print ("Message read by some participants.")
    case .delivered:
        print ("Message delivered.")
    case .delivering:
        print ("Message is delivering.")
}
```

### Marking messages as read

On the recipient client side, you need to update the message status if the message is read. For example, in `viewDidAppear` method of your message view controller.

```swift
SKYContainer.default().chatExtension?.markReadMessages(messages,
    completion: { (error) in
        if error != nil {
            print("Error on marking messages as read. " +
                  "Error: \(error.localizedDescription)")
            return
        }
        print("Messages are marked as read")
})
```

Note: `messages` is an array of `SKYMessage`.

## Message unread count

### Conversation unread count
You can show the unread count for different conversations in the conversation list.
- [`fetchUnreadCountWithConversation:completion:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)fetchUnreadCountWithConversation:completion:)

```swift
SKYContainer.default().chatExtension?.fetchUnreadCount(
    conversation: conversation,
    completion: { (dict, error) in
        if error != nil {
            print ("Unable to get the unread count. " +
                   "Error: \(error.localizedDescription)")
            return
        }
        if let unreadMessages = dict?["message"]?.intValue {
            print ("Unread count: \(unreadMessages)")
        }
})
```

### Overall unread count
You may wish to show the overall unread count of all conversations in the badge value of your app.

- [`fetchTotalUnreadCount:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)fetchTotalUnreadCount:)

```swift
SKYContainer.default().chatExtension?.fetchTotalUnreadCount(
    completion: { (dict, error) in
        if error != nil {
            print ("Unable to get the total unread count. " +
                   "Error: \(error.localizedDescription)")
            return
        }
        if let unreadMessages = dict?["message"]?.intValue {
            print ("Total unread count: \(unreadMessages)")
        }
})
```

### Resetting the unread count
The unread count can be reset by [marking the messages as read](#marking-messages-as-read).

## Typing indicator
The [`SKYChatTypingEvent`](https://docs.skygear.io/ios/chat/reference/latest/Enums/SKYChatTypingEvent.html) has these events:

- [`SKYChatTypingEventBegin`](https://docs.skygear.io/ios/chat/reference/latest/Enums/SKYChatTypingEvent.html#/c:@E@SKYChatTypingEvent@SKYChatTypingEventBegin) - User began typing
- [`SKYChatTypingEventPause`](https://docs.skygear.io/ios/chat/reference/latest/Enums/SKYChatTypingEvent.html#/c:@E@SKYChatTypingEvent@SKYChatTypingEventPause) - User stopped typing
- [`SKYChatTypingEventFinished`](https://docs.skygear.io/ios/chat/reference/latest/Enums/SKYChatTypingEvent.html#/c:@E@SKYChatTypingEvent@SKYChatTypingEventFinished) - User stopped typing and the message is sent

You can make good use of these events to implement the typing indicator feature in your app.

### Subscribing to typing indicator
Skygear Chat provides real-time update to typing indicators in a particular conversation.

[`subscribeToTypingIndicatorInConversation:handler:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)subscribeToTypingIndicatorInConversation:handler:)

```swift
SKYContainer.default().chatExtension?.subscribeToTypingIndicator(
    in: conversation,
    handler: { (indicator) in
        print("Receiving typing event")
})
```

### Sending an User's typing status
To get typing status from other devices, you should always update your typing status to the server with [`subscribeToTypingIndicatorInConversation:handler:`](https://docs.skygear.io/ios/chat/reference/latest/Classes/SKYChatExtension.html#/c:objc(cs)SKYChatExtension(im)subscribeToTypingIndicatorInConversation:handler:).

```swift
SKYContainer.default().chatExtension?.sendTypingIndicator(.begin, in: conversation)
```
```swift
SKYContainer.default().chatExtension?.sendTypingIndicator(.pause, in: conversation)
```
```swift
SKYContainer.default().chatExtension?.sendTypingIndicator(.finished, in: conversation)
```

Most app developers should call the method: `sendTypingIndicator(_:in:at:completion:)`

```swift
SKYContainer.default().chatExtension?.sendTypingIndicator(event,
    in: (conversation?conversation)!,
    at: Date()) { (error) in
        if error != nil {
            print ("Error on sending typing indicator. " +
                   "Error: \(error.localizedDescription)")
            return
        }
        print ("Sent typing indicator")
}
```
with the current date `Date()` in the `at` parameter.

## User online
Coming soon
