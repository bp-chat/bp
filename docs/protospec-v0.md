# Intro

VERSION\nInstance_CMD_ID\nCOMMAND_ID\nHMAC-SHA3-512signature\npayload
ie :
01\n123\nCNN\nSignature\nPayload

Version 2 bytes (uint16)
Command Id 1 byte (uint8)
Also 2 bytes (uint16) The command can be any specified on the list bellow
the signature is the hmc-sha3-512 hash of the payload using the account private key  (64 bytes)
the payload is dinamic based on the command, but follows the same basic structure as the header. Values separated by the a \n character.


## Commands

| Id | Name |Payload|
|---|---|---|
|0|whoo|xxxxxxxxxxxxxxxxxxxxxxxxx|
|1|whoo1|xxxxxxxxxxxxxxxxxxxxxxxx|