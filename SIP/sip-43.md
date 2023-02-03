---
sip: 43
title: Introduce Smart Token Owner
description: Introduce transferable ownership for smart tokens
author:  frank_the_tank
status: Final
Last Call: 2022-11-19
type: Standard
category: Core
created: 2022-10-09
---
## Abstract
Allow the issuer of a smart token (current owner) to transfer the ownership to another account.

## Motivation
With the goal to have more and more use cases and business cases on the chain, we should also support the buy-out/transfer of a business to a new owner. With the current setup, a smart token and its corresponding actions, like *minting* and *setting up a treasury account* are bound to the original issuer account.  

As sharing the passphrase with a new token owner has apparent drawbacks, we need to find a way to securely transfer the ownership from the former owner to a new owner. To be able to do so, we should introduce a new transaction **TransferAssetOwnership**.

With this new transaction, the current owner (by creating a smart token, the sender account of the transaction) can transfer the ownership to another account, the new token owner. After the transaction has at least **one confirmation** on the chain, the smart token owner will change to the new owner.

The following functions can only be executed by the owner:

-   Mint token
-   Set Treasury-Accounts
-   Set new owner

The minted tokens will always be added to the current owner account of the smart token.

## Specification
When a token is issued, the current *owner* is the *issuer*.
Only the current owner can execute the new **TransferAssetOwnership**, being a new transaction sub-type:
```
SUBTYPE_COLORED_COINS_TRANSFER_OWNERSHIP = 10
```

This new transaction should attach as `referencedTransactionFullHash` the full hash of the original transaction that issued the token.
Every time the token is used internally, the latest ownership transfer should be checked to verify the current owner.

The transaction should cost the same as a new creation of a smart token; 150 Signa.

## Backwards Compatibility  
This is a hard forking change, thus breaking compatibility with old fully-validating nodes.  
It should only be deployed with widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
