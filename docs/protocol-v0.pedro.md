# V0 Proposal

## Contents
1. [Glossary](#glossary)
2. [Intro](#intro)
3. [Channel](#channel)
4. [Commands](#commands)
    1. [Basics](#basics)
    2. [Users](#users)
    3. [Channels](#channels)


## Glossary

**user**: One who dare to try send a message using this
protocol, usually using a client application.

**message**: data exchanged between users.

**server**: application capable of dealing with commands,
and relay messages between users

**client**: anything that establishes a connection with a server.

## Intro

A client and server connects with each other over tcp, and 
communicates using commands.

Commands should be composed of three parts separeted by an \n character 
the command, content length, optional payload and signature i.e.:  

> "COMMAND_NAME\n4\nHMAC-SHA3-512signature\npayload "

The command can be any specified on the list bellow  
The content length should be the number of bytes of the payload  
the signature is the hmc-sha3-512 hash of the payload using the account private key  
the payload is a msgpack encoded object.

There is no response, the server may send another command related 
to the previous command, it may not, the client should not wait for 
a response.

After the initial connection, the client must send a CNN command in the first z seconds  
or the server will disconnect the client, the client must send a KALIVE command every x amount of minutes 
to keep the connection open, this amount of time may change, depending on the server configuration.

To keep it easier the basic payload representation on the list
will not add an nonce and timestamp in the samples,
but they should be present in every command to protect
the private keys.

NOTE: revoking private keys looks like a nightmare, maybe, we should always put this 
in a walled garden, and let the wall inform us who sent the command?  

## Channel

There is a lot of commands related to channels, maybe I should just split the command
section in two, so it is a little bit easier to follow.
Anyway, we don’t need to implement all of this, but, if we don’t want to keep the channel messages in the server I believe we can use a kind of distributed database strategy to keep everyone up to date. Every user is a node on this database, there will be a lot of merges, but, 
I believe we can deal with it. If the server is allowed to keep the encrypted messages than we can just treat it as a always online node and remove the at least 3 nodes limitation.
The idea of a leader is that only it should have to deal it merges.

## Commands

### Basics

#### Client to server

**CNN**  
First commands sent after the initial connection with the server  
It should contain the account identifier, a unix timestamp and a 8 bytes nonce.  
It will disconnect the client, if the validation fails.  
payload: (id:acc_id, timestamp:1702622553, nouce:15074760468938913545)

**KALIVE**  
Empty payload, should be sent every x amount of time, to keep the connection open 
if no other message was sent.

**DCNN**  
Ask the server to terminate the connection.

**INFO**  
Ask for a command with the server configuration.

#### Server to client

**SINFO**  
Current server configuration, this is the minimal payload,   
payload: (protocol_verion:0, server_name: app_name, version: 0, timeout_s: 180)

**UCNN**  
A user was just connected to the server  
payload: (id: user_id, name: nickname)

**UDCNN**  
A user was disconnected to the server  
payload: (id: user_id, name: nickname)


### Users

#### Client to server

**LST_USR**  
Request a command from the server with the current connected user list.

**CNN_USR**  
Request a handshake message to be sent to a user.  
Later it could be used to initiate a e2ee process  
payload: (to:user_id)

**PV_MSG** 
Send a message to a connected user.  
payload: (to:user_id, msg_id: ulid,  msg: some_data)

**ACK_MSG**  
Confirm that the a message has received.  
payload: (from:user_id, msg_id: ulid)

#### Server to client

**USR_HSK**  
Inform the necessary information to start a private chat with an user.  
payload: (id: user_id) //expand with e2ee and client features, if needed

**USR_ADD**  
The client may receive this message when a new user is added to the server.
The server will not send this message if the user was not connected when the action occurred.  
payload: (id: user_id, status: user_status, name: some_name)

**USR_RMD**  
The client may receive this message when a new user is removed from the server.
The server will not send this message if the user was not connected when the action occurred.  
payload: (id: user_id)

**USR_LST**  
Inform list of current known users.    
payload: (users: [(id: user_id, status: user_status, name: some_text)]) //maybe some pre uploaded auth public key?

**MSG**  
Add a message to a private chat.  
payload: (from:user_id, msg_id: ulid, msg: some_data)

**MSG_ACK**  
Confirms that a private message was received.  
payload: (msg_id: ulid)

### Channels

#### Client to server

**CH_LST**  
Request a message with a list of available channels.  

**CH_PULL**  
Request a message with an updated message history.  
The anchor message should be the last committed message that the 
current leader and the user have in common.  
payload: (id: channel_id, anchor: msg_id)

**CH_MERGE**  
Send a list of missing committed messages to the server  
payload: (msg[(id:msg_id, from: user_id, timestamp: msg_timestamp, msg: some_data, acks[user_id])])

**CH_MSG**  
Send a message to a channel.
Messages can only be sent to healthy channel  
(at least 3 online members and with an elected leader).  
payload: (id: channel_id, msg_id:ulid, msg: some_data)

**ACK_CH_MSG**  
Inform that a message has been received  
payload: (id: msg_id)

**CH_COMMIT**  
Inform that the message should be saved
on the channel history.  
Only the current leader should send this command.  
payload: (id: msg_id, acks:[user_id])

**CH_LEAD**  
Request that the server send a vote request to the current users.  
payload: (election: election_number)

**CH_VOTE**  
Request that the server inform your vote in the current election.  
payload: (election: election_number, user: user_id)

**CH_ACK_REQ**  
In case of a merge, at least one more user 
that was involved in the commit of the missed messages,
should confirm that this message existed.  
payload: (id: msg_id, users:[user_id])

### Server to client

**LST_CH**  
Inform list of current known channels.  
payload: (channels:[(id: channel_id, name:some_name, leader: usr_id,  status:channel_status)])

**CH_UJOIN**  
This message is received when someone joins a channel.  
You can receive this message with your own Id.  
payload: (channel: channel_id, user: user_id)

**CH_ULEAVE**  
This message is received when someone leaves a channel.  
You can receive this message with your own Id.  
payload: (channel: channel_id, user: user_id)

**CH_STATUS**  
The client can expect to receive this message sometime
after the connection is established, it will update the
status of a channel so the client can rejoin the conversation.   
payload: (id: channel_id, name: some_text, leader: user_id)

**CH_APPEND**  
Update the channel chat history, with a list of messages from a known anchor point.
The client is expected to send a CH_MERGE message if it has any message in the timeframe of 
the append list, and it was not in the list.  
payload: (id: channel_id, anchor: msg_id, messages:[(id: msg_id, from: usr_id, msg: some_data, timestamp: 0, acks:[usr_id])])

**CH_MSG**  
Receveid when a message was sent to a channel.  
payload: (id: channel_id, msg_id:ulid, msg: some_data)

**CH_MSG_ACK**  
Received when another user was acknowledge a message.  
payload: (id: msg_id, user: usr_id)

**CH_MSG_COMMIT**  
Receveid when a majority of user already acknowledge 
the message and it can be safely saved on the chat history.  
payload: (id: msg_id, acks:[user_id])

**CH_MSG_CONFIRM**
Receveid when the leader needs a confirmation 
that a message was sent, generally in a merge.  
The client have to send a new ACK_CH_MSG for the 
requested message, if he have receveid commited it.  
payload: (id: channel_id, message: msg_id)

**CH_ELECTED**  
Receveid when a election was finished.  
payload: (election: election_number, leader: usr_id)

**CH_VOTE**  
Receveid when somenone want you to vote for him, in a new election.    
payload: (election: election_number, user: user_id)
