---
sip: 43
title: Introduce Smart Token Owner
description: Introduce transferable ownership for smart tokens
author:  frank_the_tank
status: Review
Last Call: 2022-11-19
type: Standard
category: Core
created: 2022-10-09
---
## Abstract
Allow a creator of a smart token (current owner) to transfer the ownership to a new owner.

## Motivation
With the goal to have more and more use-cases and business-cases on the chain, we should also support the buy-out/transfer of a business to a new owner.
With the  current setup a smart token and his corresponding actions, like minting and  set Treasury-Account is bind to the creator account.  

As the passphrase can´t be changed from the creator account, we need to find a way to securley transfer the ownership from the former owner to a new owner.
To be able to do so we should introduce a new transaction **TransferAssetOwnership**.

With this new transaction, the current owner (which is by creation of a smart token = sender account of the transaction) can enable another account to be the new owner.  After the transaction is added to the chain, the smart token owner will change to the new owner.

The follwoing function can only be executed by the owner:

-   Mint token
-   Set Treasury-Accounts
-   Set new Owner

The minted tokens will always be add to the current owner account on the smart token.

## Specification
On the DB table `asset` we we will introdcue a new field `owner_id` beside the `account_id`.
By creation of a smart token the `owner_id` is equal the `account_id`.
Only the current owner set in the owner_id field can execute the new **TransferAssetOwnership** . 

All checks in the node code which are using the table `asset` and the field `account_id` need to be redirected to the new field `owner_id`.
A migration script is needed to set `owner_id` = `account_id` for all exisitng smart tokens.
By creation of a new smart token the `account_id` and `owner_id` need to be set with the sender id of the transaction.

The transaction should cost the same as a new creation of a smart token; 150 Signa.


## Backwards Compatibility  
This is a hard forking change, thus breaking compatibility with old fully-validating nodes.  
It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
