DISCLAIMER: I have little to no knowledge of encryption, so everything that's written below around this theme is mainly my imagination on how those things work.

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
