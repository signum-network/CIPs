---
sip:  6
title: Unconfirmed Tx Queue Optimizations
description: Unconfirmed Tx Queue Optimizations
author: PoCC/Brabantian
status: Final
type: Standard
category : Interface 
requires :  SIP-3, SIP-7
created: 2018-07-06
---

## Abstract
The ability to issue many transactions with a "too low" fee can cause some strain on Unconfirmed Transaction (UT) handling and UT propagation. While the current wallets can cope with it, there is still room for improvement. It is possible to estimate which UT has a chance at all to be included in a block and which hasn't. Purging UTs that have no chance of being included will lower forging load as well as network communication load.

## Motivation
UTs are transactions that have not yet been applied to the blockchain. This Unconfirmed state means that at that point of time they can not be considered final yet. There is always a possibility that these transactions will never be applied.

One of the most important reasons this could happen is because of too much "noise" on the network. This would consist of too many low cost UTs which would never be accepted due to their too low cost.

These "useless" UTs will still take RAM, processing time and bandwidth, but without any cost whatsoever. This is a possible attack vector, because it enables attackers to bombard the network with these UTs, and severely decreasing network conductivity. The solution to mitigate this, is through a smarter management of the UTs kept and propagated further through the network, while dropping the ones that never had a chance.

## Specification
The Unconfirmed Transaction Store (UTS) is a data storage that holds the current set of UTs. It offers a number of operations to add, remove, and fetch UTs so these can be processed by the system.

The goal of this CIP is to make the UTS more intelligent, allowing it to manage on its own which UTs it will keep depending on a set of rules. The way these rules will be applied will likely depend on the implementation of them. The ultimate goal is to have them applied at the earliest moment, so a "useless" transaction either never gets added, or gets kicked out
as soon as possible. This means that the UTs in the UTS should always be following the rules.

It's also important to keep in mind that certain rules need to be re-applied later on, due to changes on the blockchain through block processing.

It is recommended to have your UTS internally use a SortedMap with the maximum slot [SIP-3](sip-3.md) an UT can get into as a key (the fee divided by a quant), and the value be a list of UTs. The following line of code defines the internal storage, having a List of UnconfirmedTransactionTiming [SIP-6](sip-6.md) for each available slot. When the List for a slot doesn't exist yet when a new UT is added it should be created. Later on, when the List becomes empty after a removal, it can be deleted again.

```java
private final SortedMap<Long, List<UnconfirmedTransactionTiming>> internalStore;
```
### Rules

The UT should be pre-verified. Verification needs to be re-occur after a new Block is pushed, because it might not be valid anymore.

The UT isn't past its Deadline yet. Since UTs can be in the UTS for a while, it's required to frequently validate this again for UTs already in the store.

A configurable maximum percentage of UTs with a full hash reference to another transaction is allowed. Additional ones will be ignored.

In case there are more transactions than can be taken from a slot within the current time and the maximum expiration time possible, following ones will be ignored. The maximum number of UTs in a single slot is the slot height times 360.

In case a duplicate UT is detected, the cheapest of the duplicated UTs will be dropped. Whether UTs are duplicates depend on the type and contents of the UT.

When an UT gets added, and the amount of UTs in the UTS goes over the configurable max amount, an UT will be dropped. This will be done by sorting on Fee (ascending), Deadline (ascending) and ID (ascending).


## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
