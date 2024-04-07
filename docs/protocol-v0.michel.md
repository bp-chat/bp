# bp protocol v0

## DISCLAIMER

I have little to no knowledge of encryption, so everything that's written below around this theme is mainly my imagination on how those things work. I did
some light reading on the subject and what is outlined below seems to be a (very) rough approximation of how E2E encryption actually works.

## Glossary

CA: Certificate authority. [ssl.com - What is a Certificate Authority (CA)?](https://www.ssl.com/article/what-is-a-certificate-authority-ca/)
MITM attack: Man-in-the-middle attack. [Imperva - Man in the middle (MITM) attack](https://www.imperva.com/learn/application-security/man-in-the-middle-attack-mitm/)

## E2E encryption

`bp` will use E2E encryption. Here's a reference for that [Security Stack Exchange - What is end-to-end encryption and how to do it (correctly / securely)?](https://security.stackexchange.com/questions/230068/what-is-end-to-end-encryption-and-how-to-do-it-correctly-securely)

For E2E encryption to work, the peers communicating on the network have to exchange their public keys. One risk that's introduced with the server managing public key exchange
is the possibilty of a MITM attack. For more information, see the PKI section below.

## P2P key exchange

One (maybe) possible way for the clients to exchange keys is for them to estabilish a P2P connection. It's probably even possible to do this over TLS: [Security Stack Exchange - Can TLS be used in P2P Encryption?](https://security.stackexchange.com/questions/165949/can-tls-be-used-in-p2p-encryption). However, this does not seem to added security benefit over the
server handling key exchange. For example, in order for Alice and Bob to P2P connect to exchange keys, the server would have to provide Alice with Bob's IP and vice-versa. If
there's a MITM, they could, for example, send their own IP to Alice as if it were Bob's and vice-versa, and then could (1) get Alice's and Bob's public keys and, most importantly,
(2) send _their own_ public key to both as if it were Alice's and Bob's. In that way, the MITM could decrypt the messages Alice encrypts for Bob using the MITM key
thinking the key belongs to Bob and vice-versa.

On top of that, the P2P key exchange represents an UX issue: both clients who wish to connect have to be online at the same time.

See the section on PKI below for an attempt to make a MITM attack more difficult.

## PKI

Although at least in part a sales pitch, this article gives a good overview on the motivation behind PKI: [Keyfactor - What is PKI? A Public Key Infrastructure Definitive Guide](https://www.keyfactor.com/education-center/what-is-pki/). The main purpose of using PKI (digital certificates and CAs) is to try to make sure that when Alice sends Bob her public key,
Bob can have more confidence that it's actually Alice's public key and not a MITM's.

TODO: fix what follows below given the new information about MITM attacks.

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
