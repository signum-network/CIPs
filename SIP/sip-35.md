---
sip:  35
title:  Cashback 
description: Incentive to use own nodes for transactions
author: frank_the_tank, jjos
status: Review
Last Call: 2022-02-18
type: Standard
category: Core
created: 2022-02-01
---

# Abstract
This SIP introduces incentives for users to run their own full nodes as well as opportunities for running full nodes as a service.

# Motivation
Signum introduced with PoC+ a consensus which keeps the mining power in a most decentralized way. 
We want to do the same for our nodes, so that everyone is used to running their own node and connecting to the network. 

This makes the Signum blockchain finally stronger and more distributed over the world. To create an incentive for running its own node (even only short term), we introduce a 25% cashback on the paid fee when a transaction is signed by the corresponding node.

This would also bring up a chance for upcoming use-cases with an own node framework to benefit from their own infrastructure for the Signum network.

## Specification
The parameter *P2P.myPlatform* will be used to introduce the 25% cashback.
If a transaction is created on a node which has set a valid  *P2P.myPlatform* with a Signum address, this address (public key) will be stored on the transaction.

When the transaction gets processed on the chain, 25% of the fee will be sent to the stored cashback receiver (account ID). This fraction of the transaction fee is sent to the cashback receiver instead of the miners.

## Backwards Compatibility
This is a hard forking change, thus breaking compatibility with old fully-validating nodes. It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
