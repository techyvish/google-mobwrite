# Overview of Protocol #

MobWrite is a (mostly) symmetrical system with the server executing the same algorithms as the client. Therefore the communication between the client and the server is also (mostly) symmetrical.  Here is a typical transcript of a client talking with the server.

Client:
```
  u:fraser
  F:34:abcdef
  d:41:=200 -7 +Hello =100
  <blank>
```
Server:
```
  f:42:abcdef
  d:34:=305
  <blank>
```

In this case the client identifies itself as user 'fraser'. The client then identifies the file of interest to be 'abcdef' and that the last patches received for this file were to create server>client version 34. The client then issues a change request, in this case the first 200 characters are unchanged, the next 7 characters (which might be 'Goodbye') are deleted, the word 'Hello' is inserted, and the remaining 100 characters are unchanged.  These changes apply to client>server version 41 and would create version 42.  A blank line terminates the request.

The server responds in a similar manner. The server does not need to identify itself, so no user line is required. The server identifies the file of interest to be 'abcdef' and acknowledges receipt of client>server version 42. The server issues notification that it has no changes to contribute to the file, which it is of the opinion is currently 305 characters in length.  These changes apply to server>client version 34 and would create version 35.  A blank line terminates the request.

For a complete explanation of the theory behind the protocol, read the paper or watch the video on [Differential Synchronization](http://neil.fraser.name/writing/sync).

# Command List #

## User: `u: or U:` ##
The client identifies itself to the server. This user ID generated randomly by the browser, if one has two tabs open with the same editor, each editor will have a different ID. The user ID is not a method of identifying the human user, just the editor instance.  If the server recognises the user ID, then the session should be matched with existing data for that user, otherwise the server may create a new session.

If the user command is sent using uppercase ("U:") then the response must repeat the user ID in the reply (potentially used if there are several users multiplexed on the same connection). If the user command is sent using lowercase ("u:") then the response may omit the user ID in the reply.

The data on the user line shares the W3 specs for names and ids in HTML: must begin with a letter (`[`A-Za-z`]`) and may be followed by any number of letters, digits (`[`0-9`]`), hyphens ("-"), underscores ("`_`"), colons (":"), and periods ("."). The data may also not be longer than 500 bytes.

## File: `f: or F:` ##
The ID of the file which is to be synchronized.  Uppercase ("F:") and lowercase ("f:") are equivalent.  Following the command is an acknowledgment of the version number ('m' when issued by a client, 'n' when issued by the server) from the previously received delta or raw command (empty if no previously received version), followed by a colon (":"), then the file ID.  If the server recognises the file ID, then the session should be matched with the existing data for that file, otherwise the server may create a new file.  The comparison of the acknowledged version number and the internal state is key to determining the actions for this session:
  1. If the version number does not equal the version number of the shadow, but does equal the version number of the backup shadow, then the previous response was lost, which means the backup shadow and its version number should be copied over to the shadow (step 4) and the local stack should be cleared.  Continue.
  1. If the version number is not equal the version number of the shadow, then this means there is a memory/programming/transmission bug so the system should be reinitialized using a 'raw' command.  Do not accept any Delta commands for this file.
  1. If the version number is equal to one of the edits on the local stack, then this is an acknowledgment of receipt of those edits, which means that edit and those with smaller version numbers should be dropped.  Continue.
  1. The version number matches the shadow, proceed.

The file ID shares the W3 specs for names and ids in HTML: must begin with a letter (`[`A-Za-z`]`) and may be followed by any number of letters, digits (`[`0-9`]`), hyphens ("-"), underscores ("`_`"), colons (":"), and periods ("."). The data may also not be longer than 500 bytes.

## Delta: `d: or D:` ##
Request an edit be made to the previously specified file from the perspective of either the previously specified user (when issued by a client) or the server (when issued by the server).

If the delta command is sent using uppercase ("D:") then the new content should overwrite the existing content (used for numeric/enum content). If the delta command is sent using lowercase ("d:") then the new content should be merged with the existing content (used for text content). The response would normally be in kind.

Following the command is the version number the delta applies to (resulting in an incremented version), followed by a colon (":"), then the data.  If the version is smaller than the current shadow version of the file, then this delta must be ignored.  If the version greater than the current shadow version of the file, the system should be reinitialized using a 'raw' command.  If the version equals the current shadow version of the file, then the delta should be patched into the file and the file's version number incremented.

The data for delta is a tab-separated list of commands. The commands are identified through the first character: an equality ("="), a deletion ("-") or an insertion ("+"). Equalities and deletions are followed by the natural number of characters to keep or discard, respectively. Insertions are followed by the hexadecimal-encoded string to insert.  Implicit in this delta is the total length of the shadow; this forms the checksum.

## Raw: `r: or R:` ##
When there is unresolvable disagreement between the client and server regarding the base text (through network corruption, memory corruption or programming errors), then a delta command will likely fail due to the length checksum.  In these cases a raw command is used to transmit the entire contents of the file (in either direction), thus reestablishing the ability to delta in subsequent exchanges.

If the raw command is sent using uppercase ("R:") then the provided content should overwrite the receiver's content (used for server-to-client or for numeric/enum content or to initialize unit tests). If the raw command is sent using lowercase ("r:") then the new content should only be used to get into sync with no effect on the receiver's content (used for client-to-server text content).

Following the command is the version number of the text being sent, followed by a colon (":"), then the data.  The data for raw is a hexadecimal-encoded string.  The local stack should be dropped and the provided data and version number used for the shadow, backup shadow (if it exists) and main text (if "R:").

## Nullify: `n: or N:` ##
The ID of the file which is to be nullified.  Uppercase ("N:") and lowercase ("n:") are equivalent.  Following the command is the file ID. If the server recognises the file ID, then the file should be deleted from the server.  Deletion is distinct from saving the empty string in that after a deletion the server stores no state for the file.  No answer is expected for a nullification command.

## Message: `m: or M:` ##
Messages allow users to broadcast status information to other users.  This might allow users to see where each other's cursors are, see their idle times or simply see who else is connected.  Messages are associated with files, so a client may submit several messages in a single session.  Messages have no security in that there is nothing to stop a client from broadcasting false information.  Messages are not guaranteed delivery; one client that synchronizes once every five seconds will only see one out of five updates sent by a client that synchronizes once a second.

If the message command is sent using uppercase ("M:") then the server should reply with the currently available messages from other clients.  If the message command is sent using lowercase ("m:") then the client's messages should be submitted to the server, but the server need not answer with the messages from other clients.

The message format is JSON.  A client sends a JSON object with name/value pairs:
```
  M:{"nickname":"Fraser","useragent":"Chrome"}
```
When the server answers, it collects the most recently received JSON message object from each client and binds them together along with a client identifier:
```
  m:{"912ec803":{"nickname":"Slesinsky","useragent":"Firefox"},"68d495ab":{"nickname":"Bloom","useragent":"Opera"}}
```
The client identifiers are required so that clients can maintain persistence from one message update to the next.  For example a client might allocate a red cursor to client "912ec803" and a blue cursor to client "68d495ab".  When new messages arrive at the next synchronization, the client will be able to persist the colours if the same client identifiers are present.  The client identifiers should not be the user's ID, but rather a hash of this ID.

The client's own message object is never sent back to itself, only the messages from collaborators.  A client's message will be rebroadcast to all other clients sharing the associated file until a) the client overwrites the message with a new one, b) the client overwrites the message with a blank message, or c) the client fails to send a message for a short period of time (typically a minute or so).

The server must parse each JSON object for syntactic validity (discarding invalid messages), but otherwise does not know or care about the content.  The field names ("nickname", "useragent", etc) in the messages and their meanings are not defined by the protocol specification.  This is an area for the MobWrite community to help define and grow.

There is no requirement for a client or server to implement messages.

## Buffer: `b: or B:` ##
The JSON-P protocol only allows client->server transmissions on URL parameters.  Since these are limited to a few hundred characters, there must be a mechanism to send large blocks of data as a single transaction via multiple connections.  Buffers have a unique name and a size.  "b:mybuffer 3 2 f%3A6%3Atest%0A" would insert "f:6:test\n" into slot two of a three slot buffer.  Buffers are created whenever an insertion command is received for an unknown buffer and are deleted after a reasonable timeout or if the buffer is executed.  The parts are transmitted in any order.  Once the buffer is fully populated, it is immediately executed, the results returned, and the buffer cleared.  Mixing buffers with other commands may lead to unpredictable results.  Buffers are designed for very short-term storage, measured in a couple of seconds or less.

The data for buffer is a hexadecimal-encoded string.

There is no requirement for the client to implement a buffer, nor is there a need for a server to implement a buffer if there is no expectation of receiving different-origin JSON-P requests.

## Notes ##
  * In the interests of IE compatibility, it is recommended that the user and file data be lowercase (IE is case-insensitive -- in violation of the W3 spec). Preserving this compatibility allows HTML form elements to use the same ID as the file being shared. It is also recommended to avoid generating an ID with two consecutive hyphens ("--") since HTML comments are sensitive to this.
  * In the interests of speed and consistency, it is recommended that the hexadecimal-encoding used by delta, raw and buffer be the same as the JavaScript [encodeURI](https://developer.mozilla.org/en/Core_JavaScript_1.5_Reference/Global_Functions/encodeURI) function, with the exception of spaces. Thus the following 83 characters are preserved, while everything else is encoded: A-Z a-z 0-9 - `_` . ! ~ `*` ' ( ) ; / ? : @ & = + $ , # `[`space`]`
  * Since all data values are encoded and the user and file names are from a restricted character set, there should never be any binary characters transmitted in the protocol.  Any session containing non-whitespace characters outside of the range of x20-x7f is in violation and may be dropped.
  * Because of MobWrite's cross-platform capabilities, different clients and servers may disagree on line break formats (\r, \n, \r\n).  To prevent serious edit wars, all MobWrite implementations should convert line breaks to \n.
  * Unknown commands are simply ignored. This allows the spec to expand without breaking older implementations.
  * Delta and raw commands will be ignored if required file (and possibly user) commands have not yet been received.
  * There is no limit to the number of commands allowed. It is perfectly acceptable to change the user and/or the file within a single transaction.
  * If the final command is not followed by a newline character and a blank line, the entire session must be ignored.  Thus a connection which is truncated will not execute at all and will not leave the system in an intermediate state.
  * A blank line at the end is also required so that the receiver knows when it is safe to close the connection.

# Normal Interactions #

There are six expected interactions when synchronizing between client and server. Three are for text content, three are for numeric/enum content. When there is a disagreement noticed between the client and the server (e.g. due to memory corruption), the server always wins. The reason is that if there are 100 collaborators, the innocent 99 should not suffer data-loss if there is one bad client.  A potential future improvement would be to detect cases where there is only one client, and let that client win.

| **Type** | **Client** | **Server** | **Comments** |
|:---------|:-----------|:-----------|:-------------|
| Text, no error | d:         | d:         | Merge changes gently. |
| Number, no error | D:         | D:         | Overwrite latest content. |
| Text, server error | d:         | R:         | Overwrite client. |
| Number, server error | D:         | R:         | Overwrite client. |
| Text, client error | r:         | d:         | Overwrite client. |
| Number, client error | R:         | D:         | Overwrite client. |