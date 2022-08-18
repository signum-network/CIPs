---
sip:  41
title: Smart Tokens Distribution Changes
description: Changes how the pro-rata distribution to smart token holders work
author:   jjos, frank_the_tank
status: Review
type: Standard
category: Core
created: 2021-08-18
---
# Introduction
With the introduction of [smart token transactions](./sip-33.md) a new transaction type was included to distribute revenue (SIGNA and other
tokens) to holders of a given token.
This was implemented in a way that tokens locked in sell offers were assumed as not circulating.

By this SIP we suggest to change this logic and apply the distribution to all token holders, regardless if they are on sell offers in the
token market.

# Specific changes

Essentially we suggest the following changes:
 - The account making the distribution is excluded from the distribution
 - All other holdings, including those locked in sell offers are accounted for in the distribution

The same applies to distributions by smart contracts. The difference being, a distribution by regular transaction will account for the
holdings as they were in the previous block.
The distribution by a smart contract will use the current state (present block) for the token positions, since smart contract transactions
are applied last in a block.

## Compatibility
The change will require a hard fork.

## Backwards Compatibility
This is a hard forking change, thus breaks compatibility with old fully-validating node.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
