---
sip: 1
title: Dynamic BRS Node Capabilities
description: This document specifies a dynamic parameter space for Signum nodes
author: PoCC/rico666 <bots@cryptoguru.org>
status: Final
type: Standards Track
category: Interface
created: 2018-01-25
---

## Abstract
This document specifies a dynamic parameter space for Signum nodes, contrary to some hard-coded values of the current NRS/pre-Signum implementation. It covers computational resources available to Signum nodes, such as computational and memory capacity.

## Motivation
In the wake of the July 2017 spam attacks to the Signum network, "wallets" (more precisely: the NRS implementation) were given a hard limit of how many unconfirmed transactions to hold in memory. Before this, the number of unconfirmed transactions was not limited. This caused a potential memory DoS-vulnerability which the author(s) of the spam attack exploited: By flooding the network with thousands of transactions, nodes eventually ran out of memory and crashed.

While leaving an unlimited parameter space is a bad idea, hard-capping this parameter also does not serve the network best, because it neglects various node configurations and their various capabilities. In short: some nodes may have much more resources available than others and if a node operator decides to make these resources available, (lack of) software configuration should not prevent this. The configuration of the Singum-Node should allow to map these capabilities to the real constraints of these nodes, or those the node operators would like to provide.

## Specification

# Memory-Related Parameters

   - maximum number of unconfirmed transactions

# Network-Related Parameters

   - maximum API answer threshold

# Computation-Related Parameters

    - maximum CPU cores used
    - GPU support enabled


## Reference Implementation
 - https://github.com/PoC-Consortium/burstcoin/blob/master/conf/brs-default.properties


## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
