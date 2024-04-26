# bp Specification and Protocol - v0

## DISCLAIMER

I have little to no knowledge of encryption, so everything that's written below around this theme is mainly my imagination on how those things work. I did
some light reading on the subject and what is outlined below seems to be a (very) rough approximation of how E2E encryption actually works.

## Glossary

CA: Certificate authority. [ssl.com - What is a Certificate Authority (CA)?](https://www.ssl.com/article/what-is-a-certificate-authority-ca/)

MITM attack: Man-in-the-middle attack. [Imperva - Man in the middle (MITM) attack](https://www.imperva.com/learn/application-security/man-in-the-middle-attack-mitm/)

Slowloris attack: A sort of network attack on systems with long lived connections where the attacker fills the server with connections that either do nothing or are very slow
to act, in that way evetually exhausting the server connection capacity with useless connections. [Cloudflare - Slowloris DDoS attack](https://www.cloudflare.com/learning/ddos/ddos-attack-tools/slowloris/)

## E2E encryption

`bp` will use E2E encryption. Here's a reference for that [Security Stack Exchange - What is end-to-end encryption and how to do it (correctly / securely)?](https://security.stackexchange.com/questions/230068/what-is-end-to-end-encryption-and-how-to-do-it-correctly-securely)

For E2E encryption to work, the peers communicating on the network have to exchange their public keys. One risk that's introduced with the server managing public key exchange
is the possibilty of a MITM attack, depending on how it's implemented. For more information, see the PKI section below.

## P2P key exchange

One (maybe) possible way for the clients to exchange keys is for them to estabilish a P2P connection. It's probably even possible to do this over TLS: [Security Stack Exchange - Can TLS be used in P2P Encryption?](https://security.stackexchange.com/questions/165949/can-tls-be-used-in-p2p-encryption). However, this does not seem to bring any added security benefit
over the server handling key exchange naively. For example, in order for Alice and Bob to P2P connect to exchange keys, the server would have to provide Alice with Bob's IP and
vice-versa. If there's a MITM, they could, for example, send their own IP to Alice as if it were Bob's and vice-versa, and then could (1) get Alice's and Bob's public keys and,
most importantly, (2) send _their own_ public key to both as if it were Alice's and Bob's. In that way, the MITM could decrypt the messages Alice encrypts for Bob using the MITM
key thinking the key belongs to Bob and vice-versa.

On top of that, the P2P key exchange represents an UX issue: both clients who wish to connect have to be online at the same time.

See the section on PKI below for an attempt to make a MITM attack more difficult.

## PKI

Although at least in part a sales pitch, this article gives a good overview on the motivation behind PKI: [Keyfactor - What is PKI? A Public Key Infrastructure Definitive Guide](https://www.keyfactor.com/education-center/what-is-pki/). The main purpose of using PKI (digital certificates and CAs) is to try to make sure that when Alice sends Bob her public key,
Bob can have more confidence that it's actually Alice's public key and not a MITM's. To provide more confidence to `bp`'s users we can (1) have a digital certificate, (2)
whenever a client is trying to connect, show ours and the CA's info so the user can have more confidence that, provided their client was not tampered with, he's actually connecting
to a genuine `bp` server. We can discuss the UX of always showing this information upon connection, and how we can make this better. Maybe the client can hardcode its expectations
regarding the server's certificate subject and CA, and fail to connect if the expectation is broken.

In this day and age saying that we have to have a digital certificate for secure connection is absolutely not groundbreaking. However, hopefully what was written up until
now provides justification for why that is necessary. At least for this writer, a lot of what's written above was previously not clear.

## Variables

`$SERVER_TIMEOUT`: The time in milliseconds the client waits for the server to respond to a command with a "synchronous" character before falling back to an error state.

`$CLIENT_TIMEOUT`: The time in milliseconds the server waits for the client to send a message with a "synchronous" character before falling back to an error state and possibly
hitting the client with a strike towards banishment. The main motivation for this is Slowloris prevention.

`$SERVER_CONNECT_TIMEOUT`: The time in milliseconds the client waits for the server's public key upon connection.

`$CLIENT_CONNECT_TIMEOUT`: The time in milliseconds the server waits for the client's hashed username.

`$CLIENT_CONNECT_TIMEOUT_LIMIT`: The integer number that determines how many times the client can timeout upon connection before being banned.

## Commands

**DISCLAIMER**: The following is an assumption of how the communication will work with digital certificates.

- [ASSUMPTION] The client will start up offline, will only attempt to connect when the user either tries to sign up or log in.

### Client - sign up

If the user does not yet have an account, he can sign up through the client.

1. User chooses the option to sign up;
2. Client asks for username, passphrase and whether user wants to be broadcasted (default is false);
3. Client encrypts the chosen username with the passphrase and the client's private key [ASSUMPTION];
4. Client connects to the server. See the command `connect` below for details;
    1. Server observes and acts on `$CLIENT_TIMEOUT`;
5. Client sends command `signup:$USERNAME:$ENCRYPTED_USERNAME:$WANT_BROADCAST`;
    1. If the server takes longer than `$SERVER_TIMEOUT` to respond, the client warns the user. More work can be done around this [UX];
    2. If the username is already in use, server sends message to the client to inform of the username existence: `signup-conflict`. The client then informs the user and asks for
    another username;
    3. [OPTION] This document from here on out is going to assume the strategy outlined above of never sending the user's password to the server. This is inspired by
    [Protected Text](https://www.protectedtext.com/). At first it seems a more secure strategy. However, I wouldn't rule out, for example, the possibility of sending the
    private-key-encrypted passphrase to the server. In this latter case, the server wouldn't have knowledge of the passphrase [SECURITY].
6. Server sends message `signup-ok` to let the client know the process succeeded. It stores the $USERNAME and $ENCRYPTED_USERNAME so the user can sign in, as well as if the user
wants to be broadcasted to other users;
7. After signup [UX]:
    1. [OPTION] The user does not get automatically logged in, and has to log in separately. If that's the case, either the client or the server would have to disconnect after
    signup;
    2. [OPTION] The user gets automatically logged in;

### Client - login

1. User chooses the option to login;
2. User provides username and passphrase;
3. Client encrypts provided username with user's passphrase and private key [ASSUMPTION];
4. Client connects to the server. See the command `connect` below for details;
    1. Server observes and acts on `$CLIENT_TIMEOUT`;
5. Client sends the command `login:$USERNAME:$ENCRYPTED_USERNAME` to the server;
    1. Client observes and acts on `$SERVER_TIMEOUT`;
    2. If credentials don't match, server sends message `login:invalid` to the client and closes the connection, the clients lets the user know;
6. Server sends the message `login:ok` to let the client know the login succeeded;

- [IDEAS]
    - Connection is kept alive so it isn't strictly required for the server to send some sort of authn token to the client upon login. However, this could improve UX. For example,
    if the client does not have a token and briefly loses connection, upon reconnection the user would have to login again. The user could choose what he prefers. If they choose
    to not have to login upon reconnection, the server could send a token upon login. Would it be possible to forge the token in such a way that only that specific client could
    use it? For example, the server encrypts a part of the token with the client public key in such a way that only the client's private key can decrypt it, and when the client
    sends back the token it has to decrypt, in that way proving that it's the intended client [ASSUMPTION]. Or could some sort of signing be used?

### Client - connect

1. The client attempts connection with the server using TLS;
    1. [OPTION] Upon handshake, the client shows the server's certificate information to the user, especially information about the server _and_ the CA;
        1. [ALTERNATIVE] The client can have hardcoded expectations about the server's certificate information, and when those expectations fail, the client refuses to connect.
        If the client has been tampered with this won't work, but if that's the case all bets are off anyway;
        2. The client may cache the user acceptance of the server certificate info for a determinate amount of time. This may be useful in some situations. For example, when the
        user tries to login the client will connect to the server and show the certificate info. If the user provides the wrong passphrase by mistake, the server will one way or
        another close the connection for Slowloris prevention. When the user tries again, the client will have to connect again. It may be an UX problem if the client shows the
        certificate info again and asks for the user acceptance. If the info is the same as it is in the cache, the client may implicitly accept the certificate without asking
        again. This may be configurable.
    2. One way or another, the user or the client itself may refuse to connect;
2. [ASSUMPTION] When the client connects it already has the server's public key contained in the server's certificate and vice-versa;

### Client - ask for user broadcast

- What is the purpose of user broadcasting?
    - This is meant to serve as "bootstraping" for the platform. In other words, it's a way `bp` users can connect to each other without relying on any 3rd party. As a thought
    experiment, how would the users be able to message each other in `bp` if it was the only means of remote communication in the world without broadcasting? Of course, broadcasting
    only the username possibly won't help people identify their friends and such. We can let the users optionally provide more information about themselves so they can be found
    more easily.

The broadcast user list is paginated.

**Pre-conditions**:

- User has to be logged in.

1. Client sends message `user-broadcast` to the server;
    1. Client observes and acts upon `$SERVER_TIMEOUT`;
    2. Server assumes client is asking for the first page of users with this specific message;
2. Server sends message `user-broadcast:$USERS:$PREVIOUS_PAGE_KEY:$NEXT_PAGE_KEY` to the client;
    1. `$USERS` is a list of usernames, those from the users who opted-in to be broadcasted. The usernames are separated by the newline character: `\n`;
        1. [OPTION] In addition to the username, the server may also provide a flag that indicates whether the user is currently online. Also, the list may be ordered by
        (1) online first, (2) alphabetically;
    2. `$PREVIOUS_PAGE_KEY` is the key the client should use if the user wants to fetch the previous page of broadcasted users. This may be empty, which implies there's no previous
    page, i.e., the user is on the first page;
    3. `$NEXT_PAGE_KEY` is the key the client should use if the user wants to fetch the next page of broadcasted users. This may be empty, which implies there's no next page, i.e.,
    the user is already on the last page;
    4. This message assumes the server uses "keyset pagination". More details on [Use the index, Luke! - We need tool support for keyset pagination](https://use-the-index-luke.com/no-offset);
3. Client shows the list to the user;
4. If the user wants to change the page on the list, the client sends the message `user-broadcast:$PAGE_KEY`;
    1. `$PAGE_KEY` is the key of either the previous or the next page the server sent along with the last request for a user broadcast page;
    2. From here on out, the process is the same as described above.

### Client - cancel broadcast

User can opt-out of being broadcasted at any time. The effect will not necessarily be immediate, though. [TODO]

1. User chooses the option to cancel his broadcasting;
2. Client sends the message `cancel-broadcast` to the server;
    1. Client observes and acts upon `$SERVER_TIMEOUT`;
3. Server sends message `cancel-broadcast:ok`;
4. Client lets the user know the operation succeeded.

### Client - search broadcasted user

Client can search for a user among the broadcasted ones. [TODO]

### Client - request authorization

- [IDEAS]
    - Client can request authorization to send message to a particular user;
        - The authorizer can be from the broadcasted users list;
        - The authorizer can be specified by text. In this case, when the client emits this request the server only acknowledges it. Upon acknowledging it, the server may
        have either (1) ignored the request because the user didn't exist or the requester _is_ the authorizer, (2) stored the request to be sent later because the authorizer
        is not online, or (3) sent the request to the authorizer. This is done so a bad actor cannot easily figure out what usernames exist. Not that this would
        necessarily matter much because (1) the authorizer have to accept the request, (2) even if the bad actor could guess the user's passphrase, because the encryption is signed
        with the user's private key [ASSUMPTION] the bad actor wouldn't be able to access the user's account;
        - There should be consequences for an user who's spamming requests, especially if it's to the same user;

### Server - send authorization request

- [IDEAS]
    - To receive messages, users have to authorize the sender.

### Client - deny authorization

Once a request has been denied it is stored. The authorizer has access to the denied requests and can (1) authorize the user, (2) do not authorize, but "clear" the denial. [TODO]

### Client - clear authorization denial

### Client - authorize

### Client - send message

### Server - send message

### Client - banishment

## Alternative client ideas

With well defined specification and protocol there can be a wealth of clients aside from the official reference client that's going to be implemented. Here are some interesting (to
me) ideas in no particular order.

### Browser based

Those would take advantage of the ubiquity of web browsers:

- Lightweight JS browser client that uses the bare minimum HTTP, almost as if it were pure TCP. For example, all requests could be `POST`s with a single header that specify the
client kind and the payload could be in the standard protocol specified above. The server could identify it's that kind of client and parse the commands slightly differently;
- Lightweight JS-less browser client. Because it's JS-less it would have to use the standard way browsers communicate with servers, and because of this communication with the
server would have to be significantly different. The server could have a dual mode of operation, but maybe what would make more sense in this case would be to have an intermediary
proxy server (written in Rust?!) that talks standard HTTP, translates it to the `bp` protocol and passes it on to the main server, and vice-versa;
- Lightweight WASM client. This can be somewhat similar to the JS client. I don't know WASM but maybe it would even possible to use pure TCP and act exactly as a native client would.

## Notes

[1] This is done so one end (A) can encrypt the message with the the other end's (B) public key, so only B can decrypt it with its private key. Is that how it works?
