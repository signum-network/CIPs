---
sip:  46
title: Block signing public key
description: A new transaction type to delegate the signing of blocks
author:   jjos, frank_the_tank
status: Review
type: standard
category: SRC
created: 2022-11-17
---
## Abstract
This SIP will allow solo miners to define a different public key, specifically to sign blocks.
In case the passphrase that generates this *block signing* public key leaks, the miner can just register a new one.
This way, a miner is able to forge solo blocks without using the original account passphrase.

# Motivation
Signum offers a simple setup to start solo mining. Currently, only a full node and the miner software are needed. [BTDEX](https://github.com/btdex/btdex/) also offers a solution for solo mining.
From a security perspective, the only downside to running a solo miner is that you have to leave the passphrase in clear text either into the node's configuration or in the mining software.

With the present SIP, we would like to solve this problem with a new transaction type where the miner can delegate the signing of blocks to another public key. This *block signing* public key would be attached to this new transaction type.

## Specification
A new transaction type is to be created where the publickey of another account can be registered to sign blocks as a solo miner. If multiple transactions of this type are created for the same account, only the last one is valid. This opens the possibility for a miner to switch to another signing key if needed.
The passphrase that generates the *signing public key* would be set in the node configuration instead of the original account passsphrase.

The original account would receive the block rewards and not the account that would be accessed by the *block signing* key.
The account related to the *block signing* key, must also be unique and cannot be bind to any other account.
The use of the account related to the *block signing* key is not blocked, but not recomended.

## Backwards Compatibility  
This is a hard forking change, thus breaking compatibility with old fully-validating nodes. It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

