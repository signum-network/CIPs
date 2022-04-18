---
sip: 37
title: General smart contract upgrade
description: Extend the smart contract max pages, adjust fees, extend and add new operations
author:  jjos, frank_the_tank
status: Review
Last Call: 2022-04-24
type: Standard
category: Core
created: 2022-04-17
---
# Abstract
General smart contract upgrade allowing to create more complex applications.

## Specification

This proposal includes the following basic changes:
 - New contracts published as `Version 3`
 - Maximum number of pages is enlarged to 40 (as opposed to 20)
 - Basic step fee is adjusted to `0.001` SIGNA to be complatible with [SIP34](https://github.com/signum-network/SIPs/blob/master/SIP/sip-34.md)

The following operations are extended:
 - `MESSAGE_FROM_TX_IN_A_TO_B = 0x0309`: accept messages longer than 64 bits, A2 contains the *page* to read
 - `SEND_A_TO_ADDRESS_IN_B = 0x0405`: if multiple messages are sent in the same block, they are concatenated one after the other
 - `B_TO_ADDRESS_OF_CREATOR = 0x030b`: returns the creator of the AT ID given in B2 (creator of this contract if B2==0)

The following new operations are included:
 - `E_OP_CODE_POW_DAT = 0x19`: calculates `x^(y/1_0000_0000)` using double precision logic
 - `E_OP_CODE_MDV_DAT = 0x2c`: calculates `(x*y)/den` using big integer logic to avoid overflow
 - `CHECK_SIG_B_WITH_A = 0x0206`: checks if the signature of [AT ID, B2..4] can be verified with the message attached on tx id in A1 (page in A2) for account id in A3
 - `GET_CODE_HASH_ID = 0x030c`: get the AT's code hash ID (or of the AT id on B2 if B2!=0)
 - `GET_ACTIVATION_FEE = 0x040d`: get the AT's minimum activation fee (or of the AT id on B2 if B2!=0)
 - `PUT_LAST_BLOCK_GSIG_IN_A = 0x040e`: put the last block *generation signature* in A

Fixes the following issues: [#495](https://github.com/signum-network/signum-node/issues/495), [#548](https://github.com/signum-network/signum-node/issues/548)

## Backwards Compatibility  
This is a hard forking change, thus breaking compatibility with old fully-validating nodes.  
It should not be deployed without widespread consensus.
Contracts of previous versions run unchanged.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
