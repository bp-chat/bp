# V0 Proposal

## Glossary

**user**: One who dare to try send a message using this
protocol, usually using a client application.

**message**: data exchanged between to users.

**server**: application capable of dealing with commands,
and relay messages between users

**client**: anything that stabilish a connection with a server.

## Intro

A client and server connects with eachother over tcp, and 
communicates using commands.

Commands should be composed of three parts separeted by an \n caracter 
the command, content length, optional payload and signature i.e.:  

> COMMAND_NAME\n4\nHMAC-SHA3-512signature\npayload 

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

## List of client to server commands

#### CNN

First commands sent after the initial connection with the server  
It should contain the account identifier, a unix timestamp and a 8bytes nonce.  
It will disconnect the client, if the validation fails.  

NOTE: revoking private keys looks like a nightmare, maybe, we should always put this 
in a walled garden, and let the wall inform us who sent the command?

payload: (id:acc_id, timestamp:1702622553, nouce:15074760468938913545)

#### KALIVE

Empty payload, should be sent every x amount of time, to keep the connection open 
if no other message was sent.

#### DCNN

Ask the server to terminate the connection.

#### INFO

Ask for a command with the server configuration.

#### LST_USR

Request a command from the server with the current connected user list.

#### CNN_USR

Request a handshake message to be sent to a user.  
Later it could be used to initiate a e2ee process  
payload: (to:user_id)

#### PV_MSG

Send a message to a connected user.  
payload: (to:user_id, msg_id: ulid,  msg: some_data)

#### AK_MSG

Confirm that the a message has received.  
payload: (from:user_id, msg_id: ulid)

#### CH_LST

Request a message with a list of available channels.  

#### CH_PULL

Request a message with an updated message history.  
payload: (id: channel_id, last_msg_id: msg_id)

#### CH_MSG

Send a message to a channel.
Messages can only be sent to healthy channel, with a leader.  
payload: (id: channel_id, msg_id:ulid, msg: some_data)

#### CH_MERGE

Send a list of missing commited messages to the server  
payload: (msg[(id:msg_id, from: user_id, timestamp: msg_timestamp, msg: some_data, aks[user_id])])

#### AK_CH_MSG
Inform that a message has beem receveid  
payload: (id: msg_id)

#### CH_COMMIT

Fix the message on the channel history.  
payload: (id: msg_id, aks:[user_id])

#### CH_LEAD

Request that the server send a vote request to the current users.
payload: (election: election_number)

#### CH_VOTE

Request that the server inform your vote in the current election.  
payload: (election: election_number, user: user_id)

#### CH_AK_REQ

In case of a merge, at least one more user 
that was involved in the commit of the missed messages,
should confirm that this message existed.
payload: (id: msg_id, users:[user_id])


## List of server to client commands

#### SINFO

Current server configuration, this is the minimal payload,   
payload: (protocol_verion:0, server_name: app_name, version: 0, timeout_s: 180)

#### UCNN

A user was just connected to the server  
payload: (id: user_id, name: nickname)

#### UDCNN

A user was disconnected to the server  
payload: (id: user_id, name: nickname)

#### USR_HSK

Inform the necessary information to start a private with an user.  
payload: (id: user_id) //expand with e2ee and client features, if needed

#### USR_LST

Inform list of current known users.    
payload: (users: [(id: user_id, status: user_status, name: some_text)]) //maybe some pre uploaded auth public key?

#### MSG

Add a message to a private chat.  
payload: (from:user_id, msg_id: ulid, msg: some_data)

#### MSG_AK
Confirms that a private message was received.  
payload: (msg_id: ulid)
<!-- 
#### LST_CH

Inform list of current known channels.  


#### CH_PULL

Request a message with an updated message history.  
payload: (id: channel_id, last_msg_id: msg_id)

#### CH_MSG

Send a message to a channel.
Messages can only be sent to healthy channel, with a leader.  
payload: (id: channel_id, msg_id:ulid, msg: some_data)

#### CH_MERGE

Send a list of missing commited messages to the server  
payload: (msg[(id:msg_id, from: user_id, timestamp: msg_timestamp, msg: some_data, aks[user_id])])

#### AK_CH_MSG
Inform that a message has beem receveid  
payload: (id: msg_id)

#### CH_COMMIT

Fix the message on the channel history.  
payload: (id: msg_id, aks:[user_id])

#### CH_LEAD

Request that the server send a vote request to the current users.
payload: (election: election_number)

#### CH_VOTE

Request that the server inform your vote in the current election.  
payload: (election: election_number, user: user_id)

#### CH_AK_REQ

In case of a merge, at least one more user 
that was involved in the commit of the missed messages,
should confirm that this message existed.
payload: (id: msg_id, users:[user_id]) -->