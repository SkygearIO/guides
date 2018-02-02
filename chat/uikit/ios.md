---
title: Skygear Chat iOS UIKit
---

[[toc]]

## About UIKit
UIKit provides the `SKYChatConversationViewController`, the main UI of a conversation.
The view of `SKYChatConversationViewController` is `SKYChatConversationView`. The view consists of the following components.

1. Title Bar
2. Avatar (Outgoing)
3. Message (Outgoing)
4. Avatar (Incoming)
5. Message (Incoming)
6. Camera Button
7. Text Input
8. Voice Button

![ConversationView Overview](/assets/chat/uikit/ios/conversation-view-overview.png)


## Prerequisite
### Step 1: Add Skygear and Skygear Chat to your project

Please follow the steps at [Quick Start](https://docs.skygear.io/guides/chat/quick-start/ios/).

### Step 2: Add Skygear Chat UIKit to your project


1. add `pod 'SKYKitChat/UI', '~> 1.1'` in `Podfile`.

```bash
pod 'SKYKitChat/UI', '~> 1.1'
```

2. Save your `Podfile` and run pod install in the terminal.

```bash
pod install
```
3.  Import UIKit in your source code as follows


```obj-c
#import <SKYKitChat/SKYKitChat.h>
#import <SKYKitChat/SKYKitChat-Swift.h>
```

```swift
import "SKYKitChat.h"
```


## Creating a conversation UI

The following code block creates a `SKYChatConversationViewController` of a conversation `c`.


```obj-c
SKYChatConversationViewController* conversationViewController = [[SKYChatConversationViewController alloc] init];
conversationViewController.conversation = c; // c is a SKYConversation
[self.navigationController pushViewController:conversationViewController animated:YES];
```

```swift
let conversationViewController = SKYChatConversationViewController()
conversationViewController.converation = c // c is a SKYConversation
self.navigationController?.pushViewController(conversationViewController, animated: true)
```

## Customizations
`SKYChatConversationView` can be customized via

1. setting properties of
- `SKYChatConversationView.UICustomization()`
- `SKYChatUIModelCustomization.default()`
- `SKYChatConversationViewController`
2. implementing delegates.


## UICustomization
You can customize `SKYChatConversationView` properties in the following way.

```obj-c
[SKYChatConversationView UICustomization].messageSenderTextColor = [UIColor blackColor];
[SKYChatConversationView UICustomization].avatarHiddenForIncomingMessages = NO;
```

```swift
SKYChatConversationView.UICustomization().messageSenderTextColor = UIColor.black
SKYChatConversationView.UICustomization().avatarHiddenForIncomingMessages = false
```

The following properties are supported.
### Avatar
|Attribute|`Description`|Type|
|---------|-----------|------|
|`avatarBackgroundColor`|Background color of the avatar|Color|
|`avatarTextColor`|Text color of the avatar|Color|
|`userAvatarType`|Avatar Type, either showing initial of the username field or image|`SKYChatConversationViewUserAvatarType`, "initial" or "image"|
|`avatarHiddenForIncomingMessages`|Hide incoming message avatar if true|Boolean, default False|
|`avatarHiddenForOutgoingMessages`|Hide outgoing message avatar if true|Boolean, default True|

### Status Text & Date Time Format
|Attribute|`Description`|Type|
|---------|-----------|------|
|`messageStatusDelivering`|Delivering text|String, default "Delivering"|
|`messageStatusDelivered`|Delivered text|String, default "Delivered"|
|`messageStatusSomeRead`|Some read text|String, default "Some Read"|
|`messageStatusAllRead`|All read text|String, default "All Read"|
|`messageSentFailed`|Failed text|String, default "Failed"|
|`messageDateFormatter`|Message delivery time date formatter|`DateFormatter`|

### Title
|Attribute|`Description`|Type|
|---------|-----------|------|
|`titleDisplayType`|See below|`SKYChatConversationViewTitleOptions`|


If `default` is set, then the title bar displays `title` field value of the conversation record. If  `otherParticipants` is set,
then the title bar displays participants' usernames (excluding the current user) in the title bar.

```obj-c
[SKYChatConversationView UICustomization].titleDisplayType = SKYChatConversationViewTitleOptionsOtherParticipants;
```

```swift
SKYChatConversationView.UICustomization().titleDisplayType = .otherParticipants
```


### Message
|`attribute`|description|type|
|---------|-----------|------|
|`messageSenderTextColor`|text color of the message sender|Color|


## SKYChatUIModelCustomization
You can change `userNameField` and `userAvatarField` via `SKYChatUIModelCustomization.default()`.


```obj-c
[[[SKYChatUIModelCustomization default]
    updateWithUserNameField:@"name"]
    updateWithUserAvatarField:@"profile_pic"];
```

```swift
SKYChatUIModelCustomization.default()
    .update(userNameField: "name")
    .update(userAvatarField: "profile_pic")
```

|Attribute|`Description`|Type|
|---------|-----------|------|
|`userNameField`|Field name of the username in `user` table|String, default "username"|
|`userAvatarField`|Field name of the avatar image in `user` table|String, default "avatar"|


##### `SKYChatConversationViewController`''s properties

You may also customize `SKYChatConversationViewController`s properties before it is displayed.

```obj-c
SKYChatConversationViewController* conversationViewController = [[SKYChatConversationViewController alloc] init];
conversationViewController.conversation = c;
conversationViewController.incomingMessageBubbleColor = [UIColor redColor];
[self.navigationController pushViewController:conversationViewController animated:YES];
```

```swift
let conversationViewController = SKYChatConversationViewController()
conversationViewController.converation = c
conversationViewController.incomingMessageBubbleColor = UIColor.red
self.navigationController?.pushViewController(conversationViewController, animated: true)
```


##### Message
|`attribute`|description|type|
|---------|-----------|------|
|`incomingMessageBubbleColor`|Incoming messages background color|Color|
|`outgoingMessageBubbleColor`|Outgoing messages background color|Color|


##### Feature On/Off
|Attribute|`Description`|Type|
|---------|-----------|------|
|`shouldShowCameraButton`|Show camera button if true|Bool, default: True|
|`shouldShowVoiceMessageButton`|Show voice message button if true|Bool, default: True|
|`shouldShowTypingIndicator`|Show typing indicators if true|Bool, default: True|
|`shouldShowMessageStatus`|Show message status if true|Bool, default: true|
|`shouldShowAccessoryButton`|Show accessory button if true|Bool, default: true|



## Delegates
You may also implement functions in `SKYChatConversationViewControllerDelegate`

### Message Date

- To customize date format
```obj-c
- (NSAttributedString *) conversationViewController:(SKYChatConversationViewController *)controller dateStringAt:(NSIndexPath *)indexPath
```

```swift
optional func conversationViewController(
    _ controller: SKYChatConversationViewController,
    dateStringAt indexPath: IndexPath) -> NSAttributedString
```

- To enable or disable date

```obj-c
(bool) conversationViewController:(SKYChatConversationViewController *)controller shouldShowDateAt:(NSIndexPath *)indexPath
```

```swift
optional func conversationViewController(
    _ controller: SKYChatConversationViewController,
    shouldShowDateAt indexPath: IndexPath) -> Bool
```

### Feature On/Off

- Accessory button


```obj-c
- (bool) accessoryButtonShouldShowInConversationViewController:(SKYChatConversationViewController *)controller
```

```swift
optional func accessoryButtonShouldShowInConversationViewController(
    _ controller: SKYChatConversationViewController) -> Bool
```

- Camera button


```obj-c
- (bool) cameraButtonShouldShowInConversationViewController:(SKYChatConversationViewController *)controller
```

```swift
optional func cameraButtonShouldShowInConversationViewController(
    _ controller: SKYChatConversationViewController) -> Bool
```

- Voice message button

```obj-c
- (bool) voiceMessageButtonShouldShowInConversationViewController:(SKYChatConversationViewController *)controller
```

```swift
optional func voiceMessageButtonShouldShowInConversationViewController(
    _ controller: SKYChatConversationViewController) -> Bool
```

- Typing indicator

```obj-c
- (bool) typingIndicatorShouldShowInConversationViewController:(SKYChatConversationViewController *)controller
```


```swift
optional func typingIndicatorShouldShowInConversationViewController(
    _ controller: SKYChatConversationViewController) -> Bool
```

- Message status

```obj-c
- (bool) messageStatusShouldShowInConversationViewController:(SKYChatConversationViewController *)controller
```

```swift
optional func messageStatusShouldShowInConversationViewController(
    _ controller: SKYChatConversationViewController) -> Bool
```

### Message sending hook

- Before a message is sent

```obj-c
- (void) conversationViewController:(SKYChatConversationViewController *)controller readyToSendMessage:(SKYMessage *)message
```


```swift
optional func conversationViewController(_ controller: SKYChatConversationViewController,
                                               readyToSendMessage message: SKYMessage)
```


- After a message is sent

```obj-c
- (void) conversationViewController:(SKYChatConversationViewController *)controller finishSendingMessage:(SKYMessage *)message
```

```swift
optional func conversationViewController(_ controller: SKYChatConversationViewController,
                                               finishSendingMessage message: SKYMessage)
```

- After a message is failed
```obj-c
- (void) conversationViewController:(SKYChatConversationViewController *)controller failedToSendMessageText:(NSString * _Nonnull)text date:(NSDate * _Nonnull)date error:(NSError * _Nonnull)error
```

```swift
optional func conversationViewController(_ controller: SKYChatConversationViewController,
                                               failedToSendMessageText text: String,
                                               date: Date,
                                               error: Error)
```


### Message fetching hook
- Before message is fetched

```obj-c
- (void) startFetchingMessages:(SKYChatConversationViewController *)controller
```

```swift
optional func startFetchingMessages(_ controller: SKYChatConversationViewController)
```

- After message is fetched

```obj-c
- (void) conversationViewController:(SKYChatConversationViewController *)controller
         didFetchedMessages:(NSArray<SKYMessage *> * _Nonnull)messages
         isCached:(BOOL)isCached
```

```swift
optional func conversationViewController(_ controller: SKYChatConversationViewController,
                                               didFetchedMessages messages: [SKYMessage],
                                               isCached: Bool)
```

- After fetching is failed

```obj-c
- (void) conversationViewController:(SKYChatConversationViewController *)controller
         failedFetchingMessagesWithError:(NSError * _Nonnull)error
```

```swift
optional func conversationViewController(
    _ controller: SKYChatConversationViewController,
    failedFetchingMessagesWithError error: Error)
```

### Participant fetching hook
- After participant is fetched
```obj-c
- (void) conversationViewController:(SKYChatConversationViewController *)controller
         didFetchedParticipants:(NSArray<SKYRecord *> * _Nonnull)participants
```

```swift
optional func conversationViewController(_ controller: SKYChatConversationViewController,
                                               didFetchedParticipants participants: [SKYRecord])
```

- After fetching is failed

```obj-c
- (void) conversationViewController:(SKYChatConversationViewController *)controller
         failedFetchingParticipantWithError:(NSError * _Nonnull)error
```

```swift
optional func conversationViewController(
    _ controller: SKYChatConversationViewController,
    failedFetchingParticipantWithError error: Error)
```
