---
sip:3
title: Variable Slot-based Fees
description: Variable Slot-based Fees
author: PoCC/rico666 <bots@cryptoguru.org>
status: Final
type: Standard
category : Core
created: 2018-04-26
---


## Abstract
The slot-based transaction fee system allows for variable fees depending on the transaction load on the blockchain. As a secondary goal it prevents spamming blocks with lots of transactions with minimum fees.

## Motivation

As of December 2017, the transaction cost structure comparison between Signum and Bitcoin (1 Signa = ca. 150 Sat), shows 0.85 satoshi/Byte in the Signum network and therefore a roughly 317x higher cost for Bitcoin block space and ca. 900x higher cost for network transactions, as a Bitcoin transaction is roughly 500 bytes in size.

In Fiat, this means that at a price of around 3 US-cent per Signa, one byte of payload costs 0.017 cent in the Signum network and around 5.4 cent in the Bitcoin network. If the hardcoded tx cost for the Signum network was to remain at 1 Signa minimum, Signa would reach the same level of tx cost that Bitcoin has today at a price of 176 ∗ 5.4 = 950.4 US-cent, therefore around $9.50.

If we look at todays price levels, Signum miners were ensuring the network for a maximum payment of 91800 Signa daily. On average, the network has been doing around 5000 tx/day. So in addition to the block reward, the transaction fee reward per day, for the whole Signum network is around $150 in December 2017. In order to cope with future development of Signum price and transaction volume, a progressive tx fee structure guideline is proposes, where wallets can decide and priorize what transaction to include in the current block depending on the fill-state of the current block and memory pool backlog. The progressive approach opens the door for more on-chain scalability. Instead of limiting the max. number of transactions to 255, we could now theoretically have an open limit and solve much of the scalability issues blockchains have per se. Instead, a conservative extension of the max. number of transactions to 1020 (4-fold) is proposed.

## Specification

### Memory-Related Parameters

1.  fee quant

## Backwards Compatibility
This is a hard forking change, thus breaks compatibility with old fully-validating node. It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
