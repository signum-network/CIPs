---
sip:  46
title: Secure solo mining setup
description: Introduce a proxy pool setup for solo miner 
author:   jjos, frank_the_tank
status: Review
type: standard
category: Core
created: 2022-11-17
---
## Abstract
This SIP will allow solo miners to use a proxy pool for signing the blocks.
If the passphrase that generates this *block signing* public key leaks, the miner can register a new one.
This way, a miner can forge solo blocks without using the original account passphrase.

# Motivation
Signum offers a simple setup to start solo mining. 
Currently, only a full node and the miner software are needed. [BTDEX](https://github.com/btdex/btdex/) also provides a solution for solo mining.
From a security perspective, the only downside to running a solo miner is that you have to leave the passphrase in the clear text either in the node's configuration or in the mining software.

With the present SIP, we would like to solve this problem with the existing Reward assignment transaction and enable the node to handle proxy pools for solo miners.


## Specification
A new account must be created to function as a proxy pool on the Signum chain.
The solo miner account to which the plots for mining belong should set the Reward assignment to the newly created proxy pool account.
The assignment will be active after 4 blocks.

In the node configuration, the proxy pool's passphrase needs to be set for signing the blocks. 
This new passphrase will be placed under the config entry *RewardRecipientPassphrases* in the structure of accountId:passphrase.
The *accountId* is taken from the solo miner account.
The *passphrase* is taken from the proxy pool account.

Example:
RewardRecipientPassphrases = 1234596885:soul wing belt zoom board helm zero two four

The config entry *SoloMininingPassphrases* will be commented out.
The reward will be paid to the proxy pool account if a block gets forged.

## Backwards Compatibility  
This proposal is backwards compatible. 
The setup will only run with a node, including the changes to handle the RewardRecipientPassphrases.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

