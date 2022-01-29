---
sip:  21
title: Adjustment for Asset-Issuance fee
description: Adjustment for Asset-Issuance fee
author: frank_the_tank,  burstjack
status: Final
type: Standard
category: Core
requires : SIP-3, SIP-20
created: 2019-05-07
---
## Motivation
The Signum Asset Exchange is a built-in peer-to-peer exchange integrated into the Signum Wallet. It allows fast, secure, and decentralized trading of Sifnum Assets. Because of its decentralized nature, there’s no need for outside organizations or agencies to meddle with its affairs, resulting in improved efficiency and reduced costs.

There’s a wide range of application for potential investments or trades in Asset Exchange due to the fact that Signum Assets can be used as a token for almost anything with value.

The Signum Asset Exchange is bulletproofed and successful working since Signum creation 2014. As the fee is since inception at 1,000 Signa for the creation of an Asset which can then be traded on the Asset- Exchange we like to propose a new fee with this SIP.

## Abstract

Any user with a Signum-Account is able to access the Asset exchange via the wallet or the corresponding API calls.The payment of those actions(fee) will take place in Signa.  
As the fee is independent from a value of an Asset or the value of an order the current fee structure need to be reviewed and adjusted.

  
While in a recent hard-fork ([SIP-3](sip-3.md)) the new minimum fee of 0.00735 Signa was introduced (previous value was 1 Signa) to all types of order handling and transfer of an asset, the adaptation for the Asset Issuing was forgotten or overlooked.

Thus, in this proposal a new fee for Asset-Issuance is suggested.

## Specification

|TX  | Action |Current fee in Signa |New fee in Signa |
|----------|-------------|-------------|-------------|
|2.0|Asset-Issuance|1,000 Signa |  100 Signa |

Tx = Transaction type plus subtype on the chain


### Explanation of Action

 - **Asset-Issuance (TX 2.0)**  
This fee is to be paid when a new native Asset is created on the Signum chain. As the minimum fee is adjusted from 1 Signa to 0.00735 Signa we should also adjust the fee for this action from 1,000 to 100 Signa.


### Summary

Making the assumption of one Signa at one USD as a possible future scenario, it is clear that the current Asset-Issuance  fees can make most use-cases infeasible. We should as a competitor to the old economy be ready for scaling and attractive pricing..

## Backwards Compatibility
This is a hard forking change, thus breaks compatibility with old fully-validating node. It should not be deployed without widespread consensus.

## References

* [File Constants.java](https://github.com/signum-network/signum-node/blob/develop/src/brs/Constants.java) need to be changed.  
public  static  final  long  ASSET_ISSUANCE_FEE_NQT  =  100  *  ONE_SIGNA



## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
