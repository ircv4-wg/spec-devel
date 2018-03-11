---
Title: Introduction of Metadata
Author: Elizabeth Myers <elizabeth@interlinked.me>
Date: 11 March 2018
Status: Draft
---

# Preface
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

All examples, unless otherwise noted, use the JSON frame format specified in the JSON frame draft.

# Background
Since the beginning, IRC has many fragments of persistent data, such as TOPIC, AWAY status, NICK, etc. Adding new pieces of data involves introducing entirely new commands and often client support. This is far from ideal.

This specification aims to generalise this by bringing a concept formerly from Atheme IRC services into the protocol itself known as metadata.

Metadata are pieces of information. They are a key-to-value mapping that contain either transient or persistent mapping. A lot of metadata can be altered at-will by the client, although some is immutable.

# Alternatives
There are no known alternatives to this proposal except for the status quo, which achieves no objectives.

# Proposal
The following command is created:

* `METADATA`

This command contains the following action parameters (parameter key name `"action"`):

* `GET`; which retrieves a value
* `SET`; which sets a value
* `DELETE`; which deletes a value
* `SUBSCRIBE`; which subscribes the user to notifications in changes for the value
* `UNSUBSCRIBE`; which unsubscribes the user from notifications in changes for the value

## The `SET` action
The `SET` action MUST have both a `"key"` and `"value"` parameter.

The server MUST respond with another `METADATA` command with the `"action"` parameter set to `SET`, the keys and values sent by the client, and whether or not the action was successful.

A `SET` command MUST only succeed if the user has the rights to modify the target's metadata.

If successful, the server MUST place the metadata keys and values in some form of persistent storage, and additionally MUST overwrite any existing values if the value or values at the given key or keys already exist.

Client to server example:

```json
{"to": {"target": "Elizafox", "type": "user"}, "command": "METADATA", "parameters": {"action": "SET", "key": "status", "value": "I'm gone!"}, "id": 1}
```

Example responses:

```json
{"to": {"target": "Elizafox", "type": "user", "local": true}, "from": {"server": "irc.butthead.org", "local": true}, "command": "METADATA", "parameters": {"action": "SET", "key": "status", "value": "I'm gone!", "message": "Metadata set."}, "success": true, "id": 1}

{"to": {"target": "Elizafox", "type": "user", "local": true}, "from": {"server": "irc.butthead.org", "local": true}, "command": "METADATA", "parameters": {"action": "SET", "key": "status", "value": "I'm gone!", "message": "Could not set metadata: permission denied", "code": 400}, "success": false, "id": 1}
```

## The `GET` action
The `GET` action MUST have a `"key"` parameter.

If the key or keys do not exist, or there are no associated values with the given key or keys, or the user has no rights to retrieve the target's metadata, the server MUST return an error.

Upon success, the server MUST respond with another `METADATA` command with the `"action"` parameter set to `GET`, the key or keys requested by the client, and whether or not the action was successful. The `"value"` parameter SHALL be set if the command was successful, to the requested metadata values at the given key or keys.

Client to server example:

```json
{"to": {"target": "Elizafox", "type": "user"}, "command": "METADATA", "parameters": {"action": "GET", "key": "status"}, "id": 1}
```

Example responses:

```json
{"to": {"target": "Elizafox", "type": "user", "local": true}, "from": {"server": "irc.butthead.org", "local": true}, "command": "METADATA", "parameters": {"action": "GET", "key": "status", "value": "I'm gone!"}, "success": true, "id": 1}

{"to": {"target": "Elizafox", "type": "user", "local": true}, "from": {"server": "irc.butthead.org", "local": true}, "command": "METADATA", "parameters": {"action": "GET", "key": "status", "message": "Key not found.", "code": 404}, "success": false, "id": 1}
```

## The `DELETE` action
The `DELETE` action MUST have a `"key"` parameter.

If the key or keys do not exist, or there are no associated values with the given key or keys, or the user has no rights to delete the target's metadata, the server MUST return an error.

Upon success, the server MUST respond with another `METADATA` command with the `"action"` parameter set to `DELETE`, the key or keys requested for deletion by the client, and whether or not the action was successful. The `"value"` parameter MAY be set to the previous deleted values if the command was successful.

The associated metadata SHALL be deleted from persistent storage. Retrievals for deleted metadata MUST behave as if the entry has never existed.

Servers MAY prohibit the deletion of certain vital metadata if it would adversely affect the operation of the server.

Client to server example:

```json
{"to": {"target": "Elizafox", "type": "user"}, "command": "METADATA", "parameters": {"action": "DELETE", "key": "status"}, "id": 1}
```

Example responses:

```json
{"to": {"target": "Elizafox", "type": "user", "local": true}, "from": {"server": "irc.butthead.org", "local": true}, "command": "METADATA", "parameters": {"action": "DELETE", "key": "status"}, "success": true, "id": 1}

{"to": {"target": "Elizafox", "type": "user", "local": true}, "from": {"server": "irc.butthead.org", "local": true}, "command": "METADATA", "parameters": {"action": "DELETE", "key": "status", "message": "Key not found.", "code": 404}, "success": false, "id": 1}
```

## The `SUBSCRIBE` action
The `SUBSCRIBE` action MUST have a `"key"` parameter.

The `SUBSCRIBE` action, if successful, SHALL cause the user issuing the action to receive all future `SET` and `DELETE` operations on the metadata keys and values, regardless of the existence of such values. Such actions SHALL have the `"from"` field set to the target.

The `SUBSCRIBE` action SHALL fail if the target is the user issuing the command or has no rights to retrieve the metadata.

Users MUST be notified of all successful `SUBSCRIBE` actions on their metadata. Channels MAY be notified of successful `SUBSCRIBE` actions on their metadata.

It is RECOMMENDED clients perform a `GET` operation after performing the `SUBSCRIBE` operation to sync the present value of metadata.

Client to server example:

```json
{"to": {"target": "Elizafox", "type": "user"}, "command": "METADATA", "parameters": {"action": "SUBSCRIBE", "key": "status"}, "id": 1}
```

Example responses:

```json
{"to": {"target": "Elizafox", "type": "user", "local": true}, "from": {"server": "irc.butthead.org", "local": true}, "command": "METADATA", "parameters": {"action": "SUBSCRIBE", "key": "message": "Subscription successful"}, "success": true, "id": 1}

{"to": {"target": "Elizafox", "type": "user", "local": true}, "from": {"server": "irc.butthead.org", "local": true}, "command": "METADATA", "parameters": {"action": "SUBSCRIBE", "key": "status", "message": "Subscription failed: permission denied", "code": 400}, "success": false, "id": 1}
```

## The `UNSUBSCRIBE` action
The `UNSUBSCRIBE` action MUST have a `"key"` parameter.

The `UNSUBSCRIBE` action, if successful, SHALL cause the user issuing the action to cease reciept of any and all future `SET` and `DELETE` operations on the metadata keys and values.

The `UNSUBSCRIBE` action SHALL fail if the target is the user issuing the command, if such issuance would adversely affect the operation of the protocol, or no successful `SUBSCRIBE` operation was performed on the key or keys.

Users MUST be notified of all successful `UNSUBSCRIBE` actions on their metadata. Channels MAY be notified of successful `UNSUBSCRIBE` actions on their metadata.

Client to server example:

```json
{"to": {"target": "Elizafox", "type": "user"}, "command": "METADATA", "parameters": {"action": "UNSUBSCRIBE", "key": "status"}, "id": 1}
```

Example responses:

```json
{"to": {"target": "Elizafox", "type": "user", "local": true}, "from": {"server": "irc.butthead.org", "local": true}, "command": "METADATA", "parameters": {"action": "UNSUBSCRIBE", "key": "message": "Unsubscription done."}, "success": true, "id": 1}

{"to": {"target": "Elizafox", "type": "user", "local": true}, "from": {"server": "irc.butthead.org", "local": true}, "command": "METADATA", "parameters": {"action": "UNSUBSCRIBE", "key": "status", "message": "Unubscription failed: you are not subscribed", "code": 404}, "success": false, "id": 1}
```

# Implicit subscriptions
When joining a channel, an implicit subscription to all of its metadata keys is created, which behaves as if the `SUBSCRIBE` action was performed on all metadata keys. However, the user MUST NOT be subscribed to any privileged keys or keys they would not normally have access to; if they later gain privilege, and would normally have an implicit `SUBSCRIBE` action performed on certain keys, the server must behave as if a `SUBSCRIBE` command was issued for said metadata. Clients SHOULD be notified of this occurence, to avoid surprising the user.

Clients MUST be able to issue `UNSUBSCRIBE` actions from individual keys in a channel, unless such issuance would adversely affect the operation of the protocol or the client. If clients later so choose, they MUST be able to reissue a `SUBSCRIBE` action for keys they have previously unsubscribed from, with the usual semantics and limitations of the `SUBSCRIBE` action.

Other implicit metadata updates MAY be created by servers, but servers SHOULD notify users of such subscriptions, unless these subscriptions are an integral part of features.

# Semantics applying to all actions
The following semantics are shared between all actions.

## Errors
Errors SHALL be signalled with the top-level `"success"` key set to `false`.

Unsuccessful actions SHALL NOT have any effect on any keys, or retrieve any sensitive information.

Servers MAY throttle metadata actions, if they so choose, but MUST report said throttling as an error.

Servers MUST restrict actions on sensitive metadata, or metadata the user has no rights to perform the given action on. Servers MAY add additional restrictions. Servers MUST report all denials as errors.

## Restrictions on keys and values
Metadata keys MUST be strings. Metadata values MUST be strings, numbers, true, false, or an array containing strings.

## Multiple keys and values in one action
All actions MUST accept multiple metadata keys as an array of strings for the `"key"` element of the `"parameters"` object. If values are required for the given command, the value of the `"value"` element of the `"parameters"` object MUST be an array containing as many elements as the `"key"` array. Servers MUST assume each element of the key array corresponds to a value in the value array.

Normal restrictions on values apply to each element in both arrays. The final effect MUST be the same as if the commands had been issued individually.

## Target validity
Unless otherwise specified, all targets SHALL be valid.

When an array is passed, servers SHALL behave as if each key in the key array and its corresponding value in the value array are attempts to set values for the given keys. Servers MAY reply with arrays with multiple keys and values for such requests, but servers MUST report successful and unsuccessful actions on keys in a separate metadata frame with the `"id"` parameter set to the parameter sent by the client in the original request.

# Security issues
Sensitive metadata MUST be protected from unauthorised access by users and channels.

Certain metadata, including privleged metadata, or metadata that should not be changed due to the fundamental immutability of that attriute, MUST NOT be modified by the server, and attempts to modify this data MUST NOT succeed.

Users SHOULD be informed of the security implications of metadata, especially subscriptions. Metadata which can be subscribed to SHOULD NOT be considered sensitive.

# Backwards compatibility
TBD, likely new commands and numerics
