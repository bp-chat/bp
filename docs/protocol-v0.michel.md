**DISCLAIMER**: I have little to no knowledge of encryption, so everything that's written below around this theme is mainly my imagination on how those things work. I did
some light reading on the subject and what is outlined below seems to be a (very) rough approximation of how E2E encryption actually works.

The main problem with the way the communication happens as outlined in the following is that the server forwards the public keys to the users who wish to chat. That is, the
server has access to the key. Of course, we are well intentioned and the server code will be open source and self-hostable, however, requiring zero trust from the server is
definitely an worthwhile goal. An alternative to requiring trust in the server is to have the users connect via P2P to exchange public keys. To be quite honest, I have no idea
how feasible this would be. First, both users would have to be online at the same time for that. This is certainly an UX issue, but the client could explain the motivation behind
this to the user and could also provide a (possibly less secure) alternative: to rely on a PKI (Public key infrastructure) provider (?) for key exchange.

https://security.stackexchange.com/questions/230068/what-is-end-to-end-encryption-and-how-to-do-it-correctly-securely
https://www.keyfactor.com/education-center/what-is-pki/

TODO: finish the disclaimer (finish talking about P2P, PKI, explain what is meant by $PUBLIC_KEY below, format references)

# Assuming stateless server (no authn)

## Variables

$SERVER_CONNECT_TIMEOUT: The time in milliseconds the client waits for the server's public key upon connection.
$CLIENT_CONNECT_TIMEOUT: The time in milliseconds the server waits for the client's hashed username.
$CLIENT_CONNECT_TIMEOUT_LIMIT: The integer number that determines how many times the client can timeout upon connection before being banned.

## Client - connect

1. The client connects to the server using TLS;
2. The server sends its public key [1] in the following format: `connect:$PUBLIC_KEY`;
    1. If the server takes longer than `$SERVER_CONNECT_TIMEOUT` to send its public key, the client automatically closes the connection and notifies the user;
3. The client asks the user for a username, encrytps it with the server's public key and sends a message in the following format: `connect:$PUBLIC_KEY:$USERNAME_HASH` [1];
    1. If the client takes longer than `$CLIENT_CONNECT_TIMEOUT` to send its public key and hashed username, the server automatically disconnects the client. This is done to
    prevent a [Slowloris attack](https://www.cloudflare.com/learning/ddos/ddos-attack-tools/slowloris/);
    2. Each time the client times out, the server gives the client's IP a strike. After `$CLIENT_CONNECT_TIMEOUT_LIMIT` many strikes, the client is banned;

## Client - banishment

## Server - broadcast users

## Client - send

## Server - send

[1] This is done so one end (A) can encrypt the message with the the other end's (B) public key, so only B can decrypt it with its private key. Is that how it works?
