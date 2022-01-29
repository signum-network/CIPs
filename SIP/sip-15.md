---
sip:  15
title: RWFDS-enabled FEE_QUANT introspection and adjustment
description: RWFDS-enabled FEE_QUANT introspection and adjustment
author: frank_the_tank
status: Draft
type: Standard
categroy: Core
requires: SIP-10
created: 2018-09-30
---
## Abstract

Signum shall provide a favourable transaction fee irrespective of its performance.  A favourable price here is less than one USD-cent per payment/multi-out.

## Motivation

With implementation of the [SIP-10](sip-10.md) it will be possible for the chain to determine the value of Signa in USD and to set the cheapest slot Tx fee to one USD-cent.

This guarantees that Signa payments will be unrivalled in terms of price, regardless of the value development of Signa payments, which will enable Signa to establish itself as a global payment network.

## Specification

With introduction of SIP-10 we could automated calculate the current maximal min-fee in relation to the Signa USD value.

So at the moment we have a fee of 0.00735 Signa (FEE_QUANT) which is less than one USD-cent.

The chain would calculate from the last 360 blocks (= 1 day), the average exchange-rate Signa/USD.

If 0.00735 Signa would be more value than one USD-cent we should adjust the min fee to the equivalent of max one USD-cent. With a RWFDS, the wallet could do so automatically.

    if (trade-rate * 0.01 < 0.00735 Signa)
    then FEE_QUANT = round(trade-rate * 0.01)

FEE_QUANT can be rounded to a 8 decimal places precision

## Backwards Compatibility
This is a hard forking change, thus breaks compatibility with old fully-validating node. It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
