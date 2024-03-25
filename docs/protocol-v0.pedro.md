# V0 Proposal

## Intro

Commands should be composed of three parts separeted by an \n caracter 
the command, content length, optional body and signature i.e.:  
> COMMAND_NAME\n4\nHMAC-SHA3-512signature\nbody 

The command can be any specified on the list bellow  
The content length should be the number of bytes of the body  
the signature is the hmc-sha3-512 hash of the body using the account private key  
the body will a msgpack encoded object  

//TODO The response is a lie.

After the initial connection, the client must send a CNN command in the first 5s  
or the server will disconnect the client, the client must send a KALIVE command every x amount of minutes
to keep the connection open, this amount of time may change.

## List of server commands

### CNN

First commands sent after the initial connection with the server  
It should contain the account identifier, a unix timestamp and a 8bytes nonce.  
It will disconnect the client, if the validation fails.

### KALIVE

Empty message, should be sent every x amount of time, to keep the connection open
