---
title: Skygear Chat Data Model
---

[[toc]]

Skygear Chat has the following data models.

## Conversation
Conversation refers to the dialogue between user, just like a chatroom or channel. 

| Attributes  | Type             | Description  |
| ------ | ------------------- | ------------ |
| `title`  | <code>String</code> | Title|
| `admin_ids` | <code>JSON Array</code> | User IDs of Administrators |
| `participant_ids` | <code>JSON Array</code> | User IDs of Participants |
| `distinct_by_participants` | <code>Boolean</code> | Distinct Flag |
| `metadata` | <code>JSON Object</code> | Custom Data |
| `last_message` | <code>Message</code> | Message Record of the Last message|
| `last_message_ref` | <code>Reference</code> | Reference to the Last message|
| `last_read_message` | <code>Message</code> | Message Record of the Last Read Message|
| `last_read_message_ref` | <code>Reference</code> | Reference to the Last Read Message|
| `unread_count` | <code>Number</code> | Number of Unread Messages|

Note: `last_read_message`, `last_read_message_ref` and `unread_count`  are user-specific information.

## Message
Message refers to a single message sent by an user. A conversation is required before sending a message.

| Attributes  | Type             | Description  |
| ------ | ------------------- | ------------ |
| `attachment`  | <code>Asset</code> | Asset|
| `body` | <code>String</code> | Body |
| `metadata` | <code>JSON Object</code> | Custom Data |
| `distinct_by_participants` | <code>Boolean</code> | Distinct Flag |
| `conversation` | <code>Reference</code> | Reference to the `conversation` |
| `message_status` | <code>String</code> | Message record of the last message. Possible values: `allRead `, `someRead`, `delivered`, `delivering`|
| `seq` | <code>Sequence</code> | Unique Sequence Number|
| `revision` | <code>Number</code> | Revision Number|
| `edited_at` | <code>DateTime</code> | Last Edited Date Time|
| `edited_by` | <code>Reference</code> | Reference to the Last Edited User|
| `deleted` | <code>Boolean</code> | Deleted flag|


## Message History
`MessageHistory` contains all fields in a `Message` except

1. `MessageHistory` does not have `deleted` field and
2. `MessageHistory` contains `parent` field, referencing the original `Message`.

## Receipt
Each receipt contains the timestamps of reading and receiving a message.


| Attributes  | Type             | Description  |
| ------ | ------------------- | ------------ |
| `user`  | <code>Reference</code> | Reference to User|
| `message` | <code>Reference</code> | Reference to Message |
| `read_at` | <code>DateTime</code> | read time |
| `delivered_at` | <code>DateTime</code> | Delivery Time |
