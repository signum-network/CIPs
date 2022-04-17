---
sip: 37
title: Smart contract upgrade
description: Extend the smart contract max pages, adjust fees, and adds new operations
author:  jjos, frank_the_tank
status: Review
Last Call: 2022-04-24
type: Standard
category: Core
created: 2022-04-17
---
# Abstract
General smart contract upgrade allowing to more complex applications.

## Specification

This proposal includes the following basic changes:
 - New contracts published as Version 3
 - Maximum number of pages is enlarged to 40 (as opposed to 20)
 - Basic step fee is adjusted to `0.001` SIGNA to be complatible with SIP34

The following operations are extended:
 - `MESSAGE_FROM_TX_IN_A_TO_B`: accept messages longer than 64 bits, A2 contains the *page* to read
 - `B_TO_ADDRESS_OF_CREATOR`: returns the creator of the AT ID given in B2 (creator of this contract if B2==0)

The following new operations are included:
 - `E_OP_CODE_POW_DAT`: calculates `x^(y/1_0000_0000)` using double precision logic
 - `E_OP_CODE_MDV_DAT`: calculates `(x*y)/den` using big integer logic to avoid overflow
 - `CHECK_SIG_B_WITH_A`: checks if the signature of [AT ID, B2..4] can be verified with the message attached on tx id in A1 (page in A2) for account id in A3
 - `GET_CODE_HASH_ID`: get the AT's code hash ID (or of the AT id on B2 if B2!=0)
 - `GET_ACTIVATION_FEE`: get the AT's minimum activation fee (or of the AT id on B2 if B2!=0)
 - `PUT_LAST_BLOCK_GSIG_IN_A`: put the last block *generation signature* in A

Fixes the following issues: #495 #548

## Backwards Compatibility  
This is a hard forking change, thus breaking compatibility with old fully-validating nodes.  
It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
