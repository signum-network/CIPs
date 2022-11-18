---
sip:  46
title: Delegate block signing
description: A new tranaction to deligate the signing of blocks
author:   jjos, frank_the_tank
status: Review
type: standard
category: Core
created: 2022-11-17
---
## Abstract
This SIP will allo solo miner to define a different account to sign the blocks with it passphrase.

# Motivation
Signum offers a simple setup to start solo mining. Currently, only a full node and the miner software are needed. BTDEX also offers a solution for starting solo mining.
From a security perspective, the only downside to running a solo miner is that you have to enter the passphrase in clear text into the node's configuration.

With this SIP-46, we would like to solve the problem with a new transaction type where the miner can delegate the signing of blocks to an account it creates.

## Specification
A new transaction type is to be created where the publickey of another account can be delegated to sign the blocks as a solo miner. If multiple transactions of this type are created, only the last one is valid. This opens the possibility for a miner to switch accounts if needed.
The passphrase of the delegated account must be set in the node configuration.


## Backwards Compatibility  
This is a hard forking change, thus breaking compatibility with old fully-validating nodes. It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

