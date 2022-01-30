---
sip: 2
title: Quadruple Block Size
description: Quadruple Block Size
author: PoCC/rico666 <bots@cryptoguru.org>
status: Final
type: Standard
category : Core
created: 2018-01-26
---


## Abstract
This SIP proposes a block size growth intended to to be better able to cope with varying transactional load on the Signum blockchain and other technological improvements for the foreseeable future.

The proposed change results in a maximum block size of moderate 175KB, yet allows in combination with Multi-Out Same to serve over 19000 recipients per block.

## Motivation
The original Signum has 255 transactions per block, which is on average every 240 seconds, resulting in an effective tx capacity of 1.0625 tx/s. This is insufficient by any standard.

Raising the block size 4-fold, together with other changes in the protocoll (such as the more efficient Multi-Out transactions - see [SIP- 4](sip-4.md) will allow us to raise tx capacity up to 80 tx/s

## Specification
The original block size of Signum derives from the number of transactions per block (255) and a given size per transaction (176 bytes are taken as reference for the "Ordinary Payment" type of transaction).

This results in a block size of 255 * 176 = 44800 bytes

Any transaction requiring more than 176 bytes (such as a payment with an attached message to it) results in lowering the total tx capacity of the blockchain. Say a transaction including a message has around 1100 bytes, then there is only space for 44 such transactions in the block.

The proposed change rises this limit 4-fold, by specifying 1020 transactions instead of 255. As a direct result, the maximum block size is 1020 * 176 = 179520 bytes

Again, it's possible to add either 1020 transactions each of the size 176 bytes (ordinary payments) or - say - 149 Multi-Out Same transactions each with 128 recipients (176+1+1024 bytes per tx) each and 3 ordinary payments.

## Backwards Compatibility
This is a hard forking change, thus breaks compatibility with old fully-validating node. It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
