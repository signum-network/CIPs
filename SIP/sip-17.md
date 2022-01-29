---
sip:  17
title: Differential UT Propagation in push & pull
description: Differential UT Propagation in push & pul
author: PoCC/Brabantian
status: Final
type: Standard
categroy: Interface
created: 2018-12-01
---
## Motivation

[Differential Unconfirmed Tx Propagation](/sip-7.md) suggested a more intelligent approach in pulling Unconfirmed Transactions (UTs) through the use of timings. Peers keep an internal timestamp linked to each UT. When another Peer pulls the UTs, the current timestamp is added next to the overview of UTs. The next time the UTs are pulled from the same Peer, this timestamp can be used to filter out the ones that already were retrieved before. This has proven to work great. The amount of data retrieved through pulling is greatly diminished, saving on both on bandwidth and processing.

After post-mortem research of spam attacks on the Burst network however, it was made clear that there certainly still was room for improvement in the P2P capabilities of the Signum-Node. SIP-7 only applies to pulling of transactions, and did not sufficiently allow for differential pushing of UTs to other Peers. Because the spam attacks relied on the pushing of UTs through broadcasting and re-broadcasting, a new method has to be applied to apply differential UT Propagation in both Push and Pull scenarios.

## Abstract

Instead of a timestamp which notes the moment when a UT was added to the Unconfirmed Transaction Store, an overview of "fingerprints" will be stored. A fingerprint is a reference to a Peer that means the Peer also "knows" about the UT. Through filtering the transactions based on these fingerprints, we can make sure that we don't send the same UT twice in both pushing and pulling scenarios, or in some cases not push anything at all.

## Specification

### (Re)broadcasting Peers

The "P2P.enableTxRebroadcast" option can be enabled to automatically (re)broadcast to the Peers defined in the "P2P.rebroadcastTo" list option. Next to this static list, the P2P.sendToLimit option is used to define the maximum amount of random extra Peers that the UTs will be (re)broadcasted to when applicable.

This means that the peers in the rebroadcastTo list are the ones that should get new UTs as fast as possible, and will propagate them fast to the rest of the network. You can compare them to the backbone of the network.

### Fingerprints in UTStore

The fingerprints overview is kept in the UTStore in a HashMap, to look them up and add fingerprints in O(1) time. In case a Transaction is added to the UTStore that already was added before, the fingerprints of the Peer adding it will be automatically added to the overview.

```java
private HashMap<Transaction, HashSet<Peer>> fingerPrintsOverview = new HashMap<>();
```

### Unconfirmed Transaction pulling

"PullUnconfirmedTransactions" in the API can be called by a Peer to retrieve new UTs. After successfully giving these results to the requesting Peer, the fingerprints for this Peer will be added to the UTStore.   

Every 5 seconds the "PullUnconfirmedTransactions" of a random connected Peer will be called, to get an overview of any UTs that might not have been pushed to us (as can be the case for non-public Peers). New UTs that don't exist in the UTStore yet will be added to the store, the rest of the UTs will just have their fingerprints marked.

In case new UTs are added, a number of Peers will be retrieved which will also be pulled for their UTs. We add fingerprints to the UTStore for the Peers. Afterwards we queue up these Peers for UT Pushing.

### Unconfirmed Transaction pushing

UT Pushing will call "ProcessTransactions" in the API of another Peer. This method should only be called with UTs of which we assume that the Peer has not processed yet. This means we should not send any UTs that have the fingerprints of the Peer on them.

To make sure we don't send any extras, we do an internal queueing in our sending capabilities. The Peers queue will only accept the same Peer once at the same time, so no concurrent sending to the queue can happen. This queue has a number of consumers to it. The consumer will first call the UTStore to get an overview of UTs that haven't gotten the fingerprints of the Peer on them yet, these UTs will then be sent to the Peer. In case there are none, no sending has to happen.

### Combined effects of pulling and pushing

The combined effects of these changes will ensure that fingerprints will be added in the scenarios of pulling UTs, UTs being pulled, pushing UTs and UTs being pushed. This all should make sure that there will be lightning-fast conductivity throughout the network by pushing, while making sure that no over-feeding should happen.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
