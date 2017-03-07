---
title: Skygear Chat
---
Skygear Chat is a collection of APIs to help you build Chat apps much easier. 

## Enabling on Chat Plugin on Skygear Portal 

To start using the Chat feature, make sure you have already enabled **Chat** in the **Plug-ins** tab in your [Skygear Portal](https://portal.skygear.io). 

![Enable chat plugin on portal](/assets/common/enable-chat-plugin-on-portal.png)

To set up a new iOS project using Skygear, please refer to the [Quick Start](https://docs.skygear.io/guides/get-started/ios/).

## Creating conversations

In order to send messages, you need to create a conversation first. You can consider **conversations** as chatrooms or channels in your application.

There are two types of conversations in Skygear:
- **Direct Conversation**, support chatting between one user to another
- **Group Conversation**, support chatting among 2 or more users

### Creating direct conversations 

You can use `createDirectConversation` to create a conversation with another user. Please specify the user ID as `userID`. 

```swift
SKYContainer.default().chatExtension?.createDirectConversation(userID: userBen,
    title: "Chat with Ben",
    metadata: nil,
    completion: { (userConversation, error) in
        if error != nil {
            print ("Create direct conversation failed." +
                   "Error:\(error.localizedDescription)")
            return
        }
        
        print ("Created direct conversation")
})
```

This example shows how to create a direct chat between the current user and `userBen`.

You can also set up optional `metadata` in your conversation for custom configurations.
  
### Creating group conversations

Besides direct chats, you can also create a group conversation with 2 or more people. 

Instead of a single `userID` as the parameter, `createConversation` accepts a list of `partcipantIDs`. A new conversation will be created with the IDs given as participants.

```swift
SKYContainer.default().chatExtension?.createConversation(
    participantIDs: [userBen, userCharles, userDavid, userEllen],
    title: "Random chat group",
    metadata: nil,
    completion: { (userConversation, error) in
        if error != nil {
            print ("Create conversation failed." +
                   "Error:\(error.localizedDescription)")
            return
        }
        print ("Created Conversation")
})
```

### Creating group chat by distinct participants *(distinctByParticipants)*

By default, if you try to create conversations with same list of participants with `createConversation`. You will eventually create different conversations with the identical participants.

This may or may not be a desire behavior in your application depends on your app design.

If you want to make sure each conversation contains distinct participants, you can set `distinctByParticipants` in options to be `true`.

By default, `distinctByParticipants` is `false`.

```swift
SKYContainer.default().chatExtension?.createConversation(
    participantIDs: [userBen, userCharles, userDavid],
    title: "Our chat group",
    metadata: nil,
    adminIDs: nil,
    distinctByParticipants: true,
    completion: { (userConversation, error) in
        if error != nil {
            print ("Create conversation failed. " +
                   "Error:\(error.localizedDescription)")
            return
        }
        print ("Created conversation")
})
```

Since we have set `distinctByParticipants` to be true, upon calling the above function twice, you will receive an error as duplicated conversations.

### Setting admins
All participant will be admin unless specific in `adminIDs`

```swift
SKYContainer.default().chatExtension?.createConversation(
   participantIDs: [userBen, userCharles, userDavid],
   title: "Ben's world",
   adminIDs: [userBen],
   distinctByParticipants: false,
   completion: { (userConversation, error) in
      if error != nil {
          print("Unable to admin. " +
                "Error:\(error.localizedDescription)")
          return
      }
      
      print("Added as admin")
})
```

In this conversation, `userBen` will be the only admin.

### Fetching existing conversations
There are `SKYUserConversation` and `SKYConversation` in the `SKYChatKit`. 

In normal situation, we query `SKYUserConversation` for display and create a `SKYUserConversation` to a start conversation. User-specific information such as unread count, is available in `SKYUserConversation`. 

`SKYConversation` contains information of a conversation that is shared among all participants, such as `participantIds` and `adminIds`.

There are three methods for fetching the existing conversations:

- `fetchUserConversations(fetchLastMessage: Bool, completion: SKYChatFetchUserConversationListCompletion?)`

Fetch the all conversations which involves the current user. The parameter `fetchLastMessage` can be used for fetching the last message in the conversation.
```swift
SKYContainer.default().chatExtension?.fetchUserConversations(
    fetchLastMessage: false, 
    completion: { (conversations, error) in
    if let err = error {
        print ("Error when fetching conversations. " + 
               "Error:\(err.localizedDescription)")
        return
    }

    if let fetchedConversations = conversations {
        print("Fetched \(fetchedConversations.count) user conversations.")
    }
})
```

If you want to have more control on the conversation fetching, you may reference to the following two methods. Documentation [here](http://cocoadocs.org/docsets/SKYKitChat/0.0.1/Classes/SKYChatExtension.html).
- `fetchUserConversation(conversation: SKYConversation, fetchLastMessage: Bool, completion: SKYChatFetchUserConversationListCompletion?)`

- `SKYContainer.default().chatExtension?.fetchUserConversation(conversationID: String, fetchLastMessage: Bool, completion: SKYChatUserConversationCompletion?)`


### Leaving conversations
To leave a conversation, you can call `leave(conversationID:completion:)` on the chat extension.


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

You can add users to an existing conversation with `addParticipants(userIDs:to:completion:)`. In the following code, `userBen`, `userCharles`, `userDavid` and `userEllen` will be added to the conversation.

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
        
        print ("Users added to the conversation")                     
})
```

### Removing users from conversation 

To remove users from a conversation, you can call `removeParticipants(userIDs:from:completion:)`. In the following code, `userBen` and `userCharles` will be removed from the conversation.

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
 The specified users will be removed from the conversation record as participants. The modified conversation will be saved to the server.

### Admin 

An admin of the conversation has the following permissions:
1. add or remove participants from to conversation, 
2. add or remove admins from the conversation,
3. delete the conversation

The number of admins in a conversation is unlimited, so you may add everyone as an admin.

#### Adding admins

You can add admins to a conversation: `addAdmins(userIDs:to:completion:)`. In the following example, `userDavid` and `userEllen` is added to be the admin of the conversation.
```swift
SKYContainer.default().chatExtension?.addAdmins(
    userIDs: [userDavid, userEllen], 
    to: conversation, 
    completion: { (conversation, error) in
        if error != nil {
            print ("Unable to add the admins. " + 
                   "Error:\(error.localizedDescription)")
            return
    }
    
    print ("Admins added")
})
```
#### Removing admins
To remove admins from a conversation: `removeAdmins(userIDs:from:completion:)`. In the following example, `userDavid` and `userEllen` are no longer admins.

```swift
SKYContainer.default().chatExtension?.removeAdmins(
    userIDs: [userDavid, userEllen], 
    from: conversation, 
    completion: { (conversation, error) in
        if error != nil {
            print ("Unable to remove the admins. " + 
                   "Error:\(error.localizedDescription)")
            return
    }
    
    print ("Admins removed")
})
```
## Chat history
### Loading all messages from a conversation 
When users get into the chatroom, you may want to load the messages of the conversation. You can specify the limit of the messages in `limit` and the time constraint for the message in `beforeTime`.
- `fetchMessages(conversation:limit:beforeTime:completion:)`
```swift
SKYContainer.default().chatExtension?.fetchMessages(
    conversation: conversation,
    limit: 100,
    beforeTime: nil,
    completion: { (messages, error) in
        if error != nil {
            print ("Messages cannot be fetched. " + 
                   "Error:\(error.localizedDescription)")
        }
        
        print ("Messages fetched")
})
```

You may also use `fetchMessages(conversationID:limit:beforeTime:completion:)` for fetching the messages. Please see the documentation [here](http://cocoadocs.org/docsets/SKYKitChat/0.0.1/Classes/SKYChatExtension.html#//api/name/fetchMessagesWithConversationID:limit:beforeTime:completion:NS_SWIFT_NAME:).
## Sending messages

A message in Skygear Chat is a `SKYMessage` record. 

To send a text message, just create a `SKYMessage` and specify the `body` of your message. `addMessage(_:to:completion:)`.


Alternatively, you can also directly create a message with `createMessage(conversation:body:metadata:completion:)` or `createMessage(conversation:body:attachment:metadata:completion:)`.


The two ways of creating a message below are **equivalent**:


1. Using `SKYMessage` and then `addMessage` to the conversation
```swift
let message = SKYMessage()
message.body = "Hello!"

SKYContainer.default().chatExtension?.addMessage(message, 
    to: userConversation.conversation) { (message, error) in
        if let err = error {
            print("Send message error: \(err.localizedDescription)")
            return
        }

        if message != nil {
            print("Send message successful")
        }
}
```

2. Directly `createMessage` in a conversation
```swift
SKYContainer.default().chatExtension?.createMessage(
    conversation: userConversation.conversation, 
    body:"Hello!", 
    metadata:nil) { (message, error) in
        if let err = error {
            print("Send message error: \(err.localizedDescription)")
            return
        }

        if message != nil {
            print("Send message successful")
        }
}
```

### Plain Text 

To send a text message, just create a `SKYMessage` and specify the `body` of your message.

Then you can use the `addMessage(message:to)` send a conversation (it works for both direct conversations or group conversations).

```swift
let message = SKYMessage()
message.body = "Hello!"

SKYContainer.default().chatExtension?.addMessage(message, 
    to: userConversation.conversation) { (message, error) in
        if let err = error {
            print("Send message error: \(err.localizedDescription)")
            return
        }

        if message != nil {
            print("Send message successful")
        }
}
```

### Metadata

Besides the body of the message, you may wish to specify metadata in your message. For example, special format or color of your message. 

`metadata` can contain a JSON format object 

```swift
let message = SKYMessage()
    message.body = "Hello! See this photo!"
    message.metadata = {"text-color":"#ff0000"}

// Then send the message
```
### Files 

If you would like to send files via Skygear Chat, you can upload a file as `SKYAsset`.

```swift
let container = SKYContainer.default()!

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
Then you can send the message with an attachment.

```swift
message.attachment = uploadedAsset
SKYContainer.default().chatExtension?.addMessage(message, 
    to: conversation, 
    completion: { (message, error) in
        if error != nil {
            print ("Failed to send message. " + 
                   "Error:\(error.localizedDescription)")
            return
        }
        print ("Sent message successfully")
})
```


## Subscribing to new messages
### Subscribing to messages in a conversation

In order to get real time update of new messages, you can subscribe to a conversation with `subscribeToMessages(in:handler:)`

```swift
SKYContainer.default().chatExtension?.subscribeToMessages(
    in: userConversation.conversation, 
    handler: { (event, message) in
        print("Received message event")
})
```

### Subscribing to messages in all conversations 

Besides a specific conversation, you might want to get notified whenever there are new messages in any conversation you belong to. 

You can subscribe to all messages in your own user channel with  `subscribeToUserChannelWithCompletion(completion:))`

```swift
SKYContainer.default().chatExtension?.subscribeToUserChannelWithCompletion(
    completion: { (error) in
        print("Received message event")
})
```

## Displaying unread count

### Conversation unread count 
You can show the unread count for different conversations in the conversation list.
- `fetchUnreadCount(userConversation:completion:)`

```swift
SKYContainer.default().chatExtension?.fetchUnreadCount(
    userConversation: userConversation, 
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

- `fetchTotalUnreadCount(completion:)`

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
The `SKYChatTypingEvent` has these events:

- `SKYChatTypingEventBegin` - User began typing
- `SKYChatTypingEventPause` - User stopped typing
- `SKYChatTypingEventFinished`- User stopped typing and the message is sent

You can make good use of these events to implement the typing indicator feature in your app.

### Subscribing to typing indicator 
Skygear Chat provides real-time update to typing indicators in a particular conversation.

`subscribeToTypingIndicator(in:handler:)`

```swift
SKYContainer.default().chatExtension?.subscribeToTypingIndicator(
    in: userConversation.conversation, 
    handler: { (indicator) in
        print("Receiving typing event")
})
```

### Sending my typing status 
To get typing status from other devices, you should always update your typing status to the server with `sendTypingIndicator(_:in:)`.

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
with the current date `Data()` in the `at` parameter.

## Recipient status
Skygear Chat is helpful for displaying recipient status such as *"Delivering"*, *"Delivered"* or *"Seen"*. 

### Message status
You can make use of the following receipt status to indicate your message status.

List of `SKYChatReceiptStatus` status:
- `SKYChatReceiptStatusDelivering` - The message is being delivered, it is not yet received by the other party.
- `SKYChatReceiptStatusDelivered` - The message is delivered, but it is not read yet.
- `SKYChatReceiptStatusRead` - The message is delivered and read. 

### Subscribing to message status change

By subscribing to `SKYChatReceiptStatus` in a message, you can get the latest status of the message sent to other recipients.

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


## Push notification
### Sending push notifications to conversation participants 

We can send the push notification to particular `userIds` and these can be retrieved by accessing the attribute `participantsIds` of `SKYConversation`.

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
- [kyperoslo/Cache](https://github.com/hyperoslo/Cache)
- [aschuch/AwesomeCache](https://github.com/aschuch/AwesomeCache)

### Handling edit and delete messages

You can edit or delete `SKYMessage` records like other records. Then save it to the cloud. 

Here is an example on how you can edit the body text of the last message in a conversation:
```swift
SKYContainer.default().chatExtension?.fetchMessages(
    conversation: conversation.conversation,
    limit: 100,
    beforeTime: nil,
    completion: { (messages, error) in
        if let err = error {
            print ("Error when fetching the messages. " + 
                   "Error: \(err.localizedDescription)")
            return
        }

        if let messages = messages {
            let lastMessage = messages.last
            lastMessage?.body = "The body is changed"
            SKYContainer.default().publicCloudDatabase.save(lastMessage, 
                completion: { (record, error) in
                    if let err = error {
                        print ("Error when saving record. " + 
                               "Error: \(err.localizedDescription)")
                        return
                    }
                    print ("Updated the message")
        })
    }
})
```
