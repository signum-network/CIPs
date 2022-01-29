---
sip: 4
title: Multi-Out Transactions
description: Multi-Out Transactions
author: PoCC/rico666 <bots@cryptoguru.org>
status: Final
type: Standard
category : Core
created: 2018-04-26
---


## Abstract
There is a scalability issue with blockchains in general and it becomes increasingly more difficult to have efficient faucets or efficient way to issue a transaction and serve several people at once. Multi-Out transactions in Signum take advantage of an attachment data field which they use for specifying a list of recipients and amounts (Multi-Out) or just recipients (Multi-Out Same) keeping overhead to a minimum.

## Motivation
A regular Signum transaction (Ordinary Payment) takes at least 176 bytes of space in the block chain. This header contains various necessary information like timestamp, signature, recipient, amount etc.

This ordinary transaction can have an attachment - often used for messages - for arbitrary additional data. Multi-Out transactions use this attachment to define a very compacted list of recipients requiring only 16 byte per recipient (Multi-Out) or even 8 byte (Multi-Out Same)

Effectively the space requirements per effective transaction (sending some amount from A to B) is up to 11 times smaller (multi-out) or 22 times smaller (multi-out same) than a ordinary transaction.

With combination of dynamic transaction fees and multi-out + multi-out same transactions, the maximum transaction per second (a.k.a "tps") Signum will able to achieve has jumped from 1 tps to 80 tps.

## Specification
A Multi-Out transaction can define up to 64 recipients and the individual amounts they shall receive in 64 pairs of 8-byte data, where

```
<Id1><amountFor-Id1><Id2><amountFor-Id2>....
```

A Multi-Out Same transaction can define up to 128 recipients who will receive all the same amount (defined in the transaction header - same place as for an ordinary transaction) like this

```
<Id1><Id2><Id3>.....
```

The Ids are the regular numeric Ids of accounts. For both types the additional constraint applies that a minimum of 2 recipients is required and no recipient Ids can be duplicate.

### Memory-Related Parameters

1.  maximum multi out recipients
2.  maximum multi same out recipients


## Backwards Compatibility
This is a hard forking change, thus breaks compatibility with old fully-validating node. It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
