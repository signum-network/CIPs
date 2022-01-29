---
sip:  26
title: Extended Reed Solomon Address Format
description: Extended Reed Solomon Address Format
author:  jjos, ohager
status: Final
type: Standard
category: Interface 
created: 2021-03-26
---
## Motivation

From the very beginning, creating accounts on Signum has not been entirely intuitive, and the process suffers from the chicken-and-egg problem: For an account to be stored on the blockchain, there must be at least one transaction involving the named account. However, a problem arises from the theoretical possibility of a collision in Account ID or the Reed-Solomon representation, since the address space for all possible addresses is smaller than that of the public key. Therefore, an account without a link to the public key is considered insecure while inactive. For outgoing transactions, the public key is implicitly linked, but not necessarily for incoming transactions. But in order to execute an outgoing transaction, the owner needs some Signa to pay for the transaction fees. To do this, the owner must somehow obtain Signa, often done by asking for a small donation in the community, or by using a working faucet service. Alternatively, an incoming transaction can include the public key along with the Account id. In this case, the account will also be secured or activated. However, it is currently not user-friendly to get the public key. In this document a simplification of the account activation process is described, which is based on an optional extension of the Reed-Solomon address.

## Abstract
The idea is to extend the Reed-Solomon address representation by the public key, such that it can be implicitly linked with the public key.

## Specification
The Account IDs can be identified by the Reed-Solomon encoding in the known form

`S-XXXX-XXXX-XXXX-XXXXX`

In this proposal, it is suggested to simply append the public key to the Reed-Solomon address, resulting in the following format

`S-XXXX-XXXX-XXXX-XXXXX-YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY`

For slightly better readability and a more compact representation, it's recommended to encode with base36. Although, base64 results in  an even more compact representation than base36, this encoding is not recommended since it is case-sensitive.

`S-XXXX-XXXX-XXXX-XXXXX-ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ`

As the address prefix can be produced from the public key, the integrity of both can be checked easily. 


_Real World Example_

`S-K37B-9V85-FB95-793HN-2UCGWTUEEY66TN7RNC189PM19C4ATCEUGQV929IY1N24H0Y82Z`

## Automatic Account Activation
Sending transactions to newly created accounts using this address variant allows the accounts to be activated automatically. This significantly simplifies the entire account creation process.

## Backwards Compatibility
This is an optional extension to the address format and has no impact on the protocol.
All newer addresses are backwards-compatible.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
