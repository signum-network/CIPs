---
sip:  10
title:  Anchor Real-World Data in Blockchain
description: Anchor Real-World Data in Blockchain
author: PoCC/rico666, PoCC/ac0v
status: Stagnant
type: Standard
category : Core
requires: SIP-11
created: 2018-07-16
---
## Abstract

Anchoring so called real-world data in the Signum blockchain is essential to get an interface between the blockchain and the world outside of the blockchain. If the blockchain could record external data in it's inherently decentral trustless consensus way, this would open a whole plethora of possible applications most cryptocurrencies will never be able to do.

Recording e.g. sensor data would enable a decentral, resilient and trusted storage for archiving purposes, chain of custody proofs or decentral validation and analysis.

## Motivation

Getting real world data anchored in the blockchain reliably is surprisingly hard, which is probably why to the best of our knowledge no other cryptocurrency does it at protocol level.

The problems are manifold. Let's say we would like to store the current temperature in New York in our blockchain. How can we algorithmically decide which value is "correct"? If all nodes of the network would retrieve that information from one data point, then they might all get "the value" (provided these nodes do not DDoS that data point, provided it does not send various values to various IPs and provided it is not down already) and actually be able to get a
consensus on that.

However, if the initial data source is a central/single entity, then putting that data into a decentral trustless consensus network is moot. It's like making something out of nothing. If this central data point became corrupted (technical problem or bad actor), then the whole consenus network would be duped.

This makes it clear, that decentral consenus data can only come from decentral data. This may put some data off limits, but plenty of interesting data remain.

In order to be able to implement "Tethered Assets" (see [SIP-11](sip-11.md)), the most interesting of these data are tickers from exchanges (regular and crypto) and central banks.

## Specification

### RWFDS Format

In this SIP, we propose a "real world fiscal data section" (RWFDS) to be stored within each block of the blockchain. This should be a simple, fixed length key-value format denoting values of certain currencies and commodities in Burst (more precisely: in Planck). The total size of the block could be a modest 1024 byte, which would leave us space for 64 key value pairs and each key could have up to 8 (ASCII)-bytes identifiers.

    00000USD:000XXXXX
    00000EUR:000YYYYY
    001g Aur:000ZZZZZ

The '0' are to be understood as padding 0-bytes, the ':' is just for visualization purposes, not existant in the real data.

### Data Source Requirements

In order to be stored in a blockchain (decentral trustless consensus-based storage) the originating data has to meet certain properties:

* several independent and reliable data sources
* other consumers depend on the validity of this data too
* sufficient redundancy, so a temporary failure of some sources can be compensated

**Independence:**

Preferably we do not fetch real world data from source X and Y, where e.g. source Y fetches its data from X too. We may use dependent sources as a means of access redundancy or load balancing for the nodes in the Signum network, but not as means of data redundancy.

**Reliability:**

The data source must have a record of reliability and uptime as well as scalability (able to sustain traffic the Signum network may impose on it). Also, we should make sure the Signum network retrievals are ok with the operator and not seen as some form of DDoS.

**Non-Exclusiveness:**

Other consumers apart from the Signum network also depend on the data. This way we make sure disruption of the service motivates also other interest groups to work towards restoring the service.

**Redundancy:**

Outage of M out of N data sources does not disrupt Signum RWDS building operation.


### Node Behavior At Source Failure

In order to ensure consensus, but to prevent excessive forking events in the case of failures, each single node in the Signum network has to adhere to the following principles:

* if a data source becomes unavailable, assume value of the past block
* if the value change of retrieved data exceeds the variance of past N blocks, assume value of the past block
* if any of the adverse events above happens, do not use the data to sign/validate a block, but only to validate data from the majority of nodes claiming to have genuine RWFDS content

If no node majority can be found, use the last RWFDS that had consensus. (Ultimate Fallback)

## Backwards Compatibility
This is a hard forking change, thus breaks compatibility with old fully-validating node. It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
