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
The `SET` action MUST have both of the following when the client sends a frame to the server:

* A key parameter, which MUST be a string or array of strings (parameter key name `"key"`)
* A value, which MUST be either an array of strings, an array containing arrays of strings, or a string (parameter key name `"value"`)

Other keys MAY be set, but SHALL NOT affect the metadata returned by the server.

When multiple keys are specified, values MUST be retrieved sequentially from the value of `"value"` if is an array. Servers MUST report an error if there are insufficient values to satisfy the request in the `"value"` field. If an array is given as keys and values, the client SHALL respond with the array of keys in `"key"` in the order sent by the client followed by values for each element in an array in `"value"`. If one of the elements of the `SET` action fails when an array is given as `"key"`, the failure MUST be signalled via a separate frame with the failed keys, and the failed keys MUST NOT be reported as successful.

The server MUST respond with `SET` as the action key followed by the status and value keys sent by the client, and whether or not the action was successful. A server MAY send a message with the response or any other keys, but said keys MUST NOT change the semantics of the `SET` action. Errors SHALL be signalled with the top-level `"success"` key set to false.

If successful, the server MUST place the metadata in some form of persistent storage. Calls to `GET` made after `SET` MUST NOT return any differing metadata, unless a `SET` action was issued by the server to all connected clients affected by this change notifying them of the new key and value set, or the `SET` action failed.

If successful, a `SET` action issued MUST replace any existing values at the given key, unless the metadata is immutable or the user has no permission to set the metadata in question; in which case the server MUST NOT overwrite the existing entry, and MUST report an error to the client.

Servers MUST NOT allow metadata to be set for other users, or on targets where the user does not have permission. Servers MAY add additional restrictions. Servers MUST report these denials as an error.

Servers MAY throttle setting of metadata, if they so choose, but MUST report said throttling as an error.

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
The `GET` action SHALL retrieve a metadata entry for the client; that is to say, the value at the given key. The `GET` action SHALL require at least one key (parameter key name `"key"`). The value for the key MUST be either an array of strings, or a string. Other keys MAY be set, but SHALL NOT affect the metadata returned by the server. Errors SHALL be signalled with the top-level `"success"` key set to false.

If an array is given as keys and values, the client SHALL respond with the array of keys in `"key"` in the order sent by the client followed by values in `"value"` given by the client. If one of the elements of the `GET` action fails when an array is given as `"key"`, the failure MUST be signalled via a separate frame with the failed keys, and the failed keys MUST NOT be reported as successful.

If the given key exists, the server MUST return the value for the given key. If the key does not exist, the server MUST return an error.

Servers MAY throttle retrieval of metadata, if they so choose, but MUST report said throttling as an error.

Servers MAY add restrictions to metadata retrieval, but servers MUST restrict sensitive metadata. Servers MUST report these denials as an error.

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
The `DELETE` action SHALL delete the given value at the given key, if such a value exists. The `DELETE` action SHALL require at least one key (parameter key name `"key"`). This key MUST be a string, or array of strings for keys to delete. Other keys may be specified as parameters, but said keys MUST NOT change the semantics of the `DELETE` action. Errors SHALL be signalled with the top-level `"success"` key set to false.

If an array is given as keys and values, the client SHALL respond with the array of keys in `"key"` in the order sent by the client. Deleted values MAY be sent back. If one of the elements of the `DELETE` action fails when an array is given as `"key"`, the failure MUST be signalled via a separate frame with the failed keys, and the failed keys MUST NOT be reported as successful.

If the given key exists, and the user has the rights to delete the key, the server MUST return the name of the key deleted. If the key does not exist, the server MUST return an error.

Servers MUST NOT allow metadata to be deleted for other users, or on targets where the user does not have permission. Servers MAY add additional restrictions. Servers MUST report these denials as an error.

Servers MAY throttle retrieval of metadata, if they so choose, but MUST report said throttling as an error.

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
The `SUBSCRIBE` action SHALL subscribe the user issuing the action to updates in certain metadata entries from the target in `"to"`. Users MAY NOT subscribe to updates from themselves. All other targets SHALL be valid. The `SUBSCRIBE` action SHALL require at least one key (parameter name `"key"`). The value for the key MUST be either an array of strings, or a string. Other keys MAY be set, but SHALL NOT affect the metadata returned by the server. Errors SHALL be signalled with the top-level `"success"` key set to false.

If an array is given as keys and values, the client SHALL respond with the array of keys in `"key"` in the order sent by the client. If one of the elements of the `SUBSCRIBE` action fails when an array is given as `"key"`, the failure MUST be signalled via a separate frame with the failed keys, and the failed keys MUST NOT be reported as successful.

`SUBSCRIBE` SHALL work even if the keys are not yet set on the target.

Once metadata is successfully subscribed to, servers MUST send all `SET` and `DELETE` actions on the metadata keys the user is subscribed to, with the `"from"` field set to the subscribee.

Subsequent calls to `SUBSCRIBE` MUST NOT affect previous or future subscriptions in any way.

Servers MAY add restrictions to metadata subscriptions, but servers MUST restrict sensitive metadata. Servers MUST report these denials as an error.

Users MUST be notified of all new subscriptions to their metadata.

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
The `UNSUBSCRIBE` action SHALL unsubscribe the user issuing the action from updates in certain metadata entries from the target in `"to"`. Users MAY NOT unsubscribe to updates from themselves. All other targets SHALL be valid. The `UNSUBSCRIBE` action SHALL require at least one key (parameter name `"key"`). The value for the key MUST be either an array of strings, or a string. Other keys MAY be set, but SHALL NOT affect the metadata returned by the server. Errors SHALL be signalled with the top-level `"success"` key set to false.

If an array is given as keys and values, the client SHALL respond with the array of keys in `"key"` in the order sent by the client. If one of the elements of the `UNSUBSCRIBE` action fails when an array is given as `"key"`, the failure MUST be signalled via a separate frame with the failed keys, and the failed keys MUST NOT be reported as successful.

`UNSUBSCRIBE` SHALL report an error if the user is not subscribed to the given key.

Once metadata is successfully unsubscribed to, servers MUST cease sending `SET` and `DELETE` actions on the metadata keys the user has unsubscribed from.

Subsequent calls to `UNSUBSCRIBE` MUST NOT affect previous or future subscriptions in any way, nor affect anything but the targeted subscriptions.

Users MUST be notified of all unsubscriptions from their metadata.

Servers MAY prohibit certain keys from being unsubscribed from, if such functionality is necessary for a feature to work.

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
When joining a channel, an implicit subscription to all of its metadata keys is created, which behaves as if the `SUBSCRIBE` action was performed on all metadata keys. However, the user MUST NOT be subscribed to any privileged keys or keys they would not normally have access to; if they later gain privilege, an implicit subscription MUST be created.

Clients MUST be able to unsubscribe from individual keys in a channel. If clients later so choose, they MUST be able to reissue a `SUBSCRIBE` action for keys they have previously unsubscribed from, with the usual semantics and limitations of the `SUBSCRIBE` action.

Other implicit metadata updates MAY be created by servers, but servers SHOULD notify users of such subscriptions, unless these subscriptions are an integral part of features.

# Security issues
Sensitive metadata MUST be protected from unauthorised access by users and channels.

Certain metadata, including privleged metadata, or metadata that should not be changed due to the fundamental immutability of that attriute, MUST NOT be modified by the server, and attempts to modify this data MUST NOT succeed.

Users SHOULD be informed of the security implications of metadata, especially subscriptions. Metadata which can be subscribed to SHOULD NOT be considered sensitive.

# Backwards compatibility
TBD, likely new commands and numerics
