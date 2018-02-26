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

```python
from chat.hooks import *
from skygear.transmitter.encoding import deserialize_record
@after_message_sent
def after_message_sent_hook(message, conversation, participants):
    message = deserialize_record(message)
    conversation = deserialize_record(conversation)
    print('Message [%s] is sent to conversation [%s].' % (message['body'], conversation['title']))
```

Note: Currently only one of the same kind hooks can be registered.

## Skygear Cloud Function
Please refer to [Introduction to Cloud Functions](https://docs.skygear.io/guides/cloud-function/intro-and-deployment/) for the basics of cloud function.

## Available Event Hooks
Event hooks are implemented in [Skygear Chat plugin](https://github.com/skygeario/chat) project. Therefore, you need to import `chat.hooks` in order to use event hooks in cloud function.

The followings are the available event hooks.

- `after_message_sent`: to run after a message is sent
- `after_message_updated`: to run after a message is updated
- `after_message_deleted`: to run after a message is deleted
- `typing_started`: to run after typing is started
- `after_conversation_created`: to run after a conversation is created
- `after_conversation_updated`: to run after a conversation is update
- `after_conversation_deleted`: to run after a conversation deleted
- `after_users_added_to_conversation`: to run after one or more users are added to a conversation
- `after_users_removed_from_conversation`: to run after one or more users are removed from the conversation

All decorated functions are not required to return any value.


### Decorated Function Interfaces and Parameters

Depending on decorators, decorated functions contain any of the following parameters. All passing values are serialized values, therefore you need to call `derserialize_record` from `skygear.encoding` to retrieve corresponding Skygear record in python.

| Tables        | Description            |
| ------------- |:----------------------:|
| `message`     | Message Record         |
| `conversation`| Conversation Record    |
| `participants`| Array of User Record   |
| `events`      | Array of Typing Events |
| `new_users`   | Array of User Records  |
| `old_users`   | Array of User Records  |


#### Function Interfaces
Below are the list of function interfaces of each decorator.

##### `after_message_sent`
```python
@after_message_sent
def after_message_sent_hook(message, conversation, participants):
    pass
```
##### `after_message_updated`
```python
@after_message_updated
def after_message_updated_hook(message, conversation, participants):
    pass
```
##### `after_message_deleted`
```python
@after_message_deleted
def after_message_updated_deleted_hook(message, conversation, participants):
    pass
```
##### `typing_started`
```python
@typing_started
def typing_started_hook(conversation, participants, events):
    pass
```
##### `after_conversation_created`
```python
@after_conversation_created
def after_conversation_created_hook(conversation, participants):
    pass
```
##### `after_conversation_updated`
```python
@after_conversation_updated
def after_conversation_updated_hook(conversation, participants):
    pass
```
##### `after_conversation_deleted`
```python
@after_conversation_deleted
def after_conversation_deleted_hook(conversation, participants):
    pass
```
##### `after_users_added_to_conversation`
```python
@after_users_added_to_conversation
def after_users_added_to_conversation_hook(conversation, participants, new_users):
    pass
```
##### `after_users_removed_from_conversation`
```python
@after_users_removed_from_conversation
def after_users_removed_from_conversation_hook(conversation, participants, old_users):
    pass
```


## Real Life Example: Push Notification
The following example demonstrates push notification implementation with Skygear API and Skygear Chat event hook. Function `after_message_sent_hook` is called after a message is sent. `participants` and `message` are then deserialized with `deserialize_record`. `other_user_ids` and `notification` are evaluated from `participants` and `message` respectively. Lastly, `push_users` is called and participants' devices receive notifications.

```python
from skygear.transmitter.encoding import deserialize_record
from skygear.action import push_users
from skygear.container import SkygearContainer
from skygear.options import options as skyoptions
from skygear.utils.context import current_user_id
from chat.decorators import *


@after_message_sent
def after_message_sent_hook(message, conversation, participants):
    container = SkygearContainer(api_key=skyoptions.masterkey,
                                 user_id=current_user_id())
    message = deserialize_record(message)
    conversation = deserialize_record(conversation)
    participants = [deserialize_record(p) for p in participants]
    other_user_ids = []
    current_user = None
    for p in participants:
        if p.id.key == current_user_id():
            current_user = p
        else:
            other_user_ids.append(p.id.key)
    content = ''
    if 'body' in message:
        content = current_user['username'] + ": " + message['body']
    else:
        content = current_user['username'] + " sent you a file."
    notification = {'gcm': {
                       'notification': {
                           'title': conversation['title'],
                           'body': content
                       }
                    },
                    'apns': {
                        'aps': {
                            'alert': {
                                'title': conversation['title'],
                                'body': content
                            },
                            'from': 'skygear',
                            'operation': 'notification'
                        }
                    }
                   }
    push_users(container, other_user_ids, notification)
```


## See Also
[Demo project](https://github.com/skygear-demo/cloud-chat-demo-py)
