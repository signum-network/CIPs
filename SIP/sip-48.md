---
sip:  48
title: Modernize the Alias framework
description: Feature enhancement for Aliases
author:   frank_the_tank
status: Review
type: standard
category: Core
created: 2022-12-10
updated: 2023-02-12
---

## Abstract
This SIP will modernize the current Alias setup. It will introduce a new valid character for the creation of Aliases and the possibility of using TLDs (Top Layer Domains) as new namespaces for an Alias and a renewal system for Aliases comparable to current worldwide web domain handling. All aliases registered before activating this SIP will stay with the current owner. Only renew them once the owner sells them.


## Motivation
Aliases have been available on the Signum chain since the genesis block.Any user could create those Aliases with one transaction.<br>
Unfortunately, the use cases were limited to those Aliases, mainly used as account-address pointers.

Besides that, the creation of an Alias is a combination of letters and numbers only.<br>
It has one namespace only and nothing like TLDs (Top Level Domain).

With the introduction of [SIP-44](https://github.com/signum-network/SIPs/blob/master/SIP/sip-44.md), the Alias will be more beneficial for domain, account or application pointers, making a reserved alias name more valuable. In addition, with [SIP-47](https://github.com/signum-network/SIPs/blob/master/SIP/sip-44.md), a new service SNS (Signum Naming Service) is in the making, a distributed, open, and extensible naming system based on the Signum blockchain. 

Given the current development and possible use cases, we need to adjust the Alias framework and modernize it to fit future requirements.

The following parts will be adjusted or introduced:

### Alias creation

Currently, it is only possible to create an Alias with a combination of letters a-z and numbers 0-9.<br>
With this SIP, we will add one extra character: _<br>
The maximum length is not changed, being 100 characters.<br>
It empowers the user to create aliases in a way they use them on other social networks or emails.


### STLD (Signum Top Level Domains)

With the current framework, the Alias system has only one dimension.<br>
Compared to an URL, we should also have a framework which allows the user to register the Alias as a second-level domain and decide in which top-level domain (TLD) the Alias should exist. 

Signum will provide the following STLD by default:

- blockhain
- coin
- crypto
- dao
- decentral
- dex
- free
- nft
- p2p
- sig
- signa
- sns
- w3
- wallet
- web3
- x
- y
- z

Aliases created without an STLD (e.g. all Aliases created before this SIP activates) have the .signum STLD for free. 


Example:

Suppose a user creates an Alias without an STLD selection like “*Spaceship*”. In that case, this Alias gets registered for “*Spaceship*” and “*Spaceship.signum*”


Suppose a user created an Alias “*Spaceship*” with the STLD, for example, “*crypto*”. In that case, this Alias is now only registered for "*Spaceship.crypto*”.



### Custom STLD
Besides the default STLDs, any Signum account can create a new STLD on the Signum chain. This way, specific use or business cases can build their namespaces and benefit from those. As Aliases have a quarterly renewal cost, the chain will send those payments to the owner of the STLD.
The following new transaction types are needed:

- **Register a new STLD**
  - An STLD must have a unique name
  - An STLD consists of a combination of ^[a-zA-Z0-9]{1,40}$ with a maximum of 40 chars
  - The minimum transaction fee is 0.01 Signa
  - The minimum amount to pay is 100k Signa (one hundred thousand)
  - The receiver account is the Zero (burn) account 
- **Sell an STLD public**
  - Similar to a public sale of an Alias
- **Sell an STLD private**
  - Similar to a private sale of an Alias

### Alias renewal process
With this SIP, every Alias becomes bound to a renewal subscription. Every time a new alias is registered (or if the owner sells a legacy alias after the upgrade), a subscription is created with a deadline of 3 months (adding 7776000 seconds to the current timestamp) and a value of 12.5 Signa. This subscription works similarly to a regular Signum subscription. When it expires, the payment gets executed, and a new deadline gets set.

Suppose the current alias owner does not have enough balance when payment for the renewal subscription gets executed.<br>
In that case, the related Alias is deleted along with its renewal subscription, making expired aliases available again for any user.

Suppose the user cancels the alias renewal subscription.<br>
In that case, the Alias is also deleted and becomes available for other users to create as fresh and new. 

Suppose an alias that already has a subscription assigned to it gets sold.<br>
In that case, the chain transfers the subscription to the new alias owner with the current deadline.

If the Alias gets deleted, the chain will automatically cancel any active private or public sale for this Alias. 

If the Alias has no TLD set (but using the default .signum) or is one of the default STLDs, the amount paid by the renewal transaction gets burned along with the subscription fee. If an Alias uses a custom STLD, the renewal amount gets distributed to the current owner of the STLD, and the subscription fee gets burned.

#### Transaction Set Alias
If a user sends a setAlias transaction, it will execute the following:
- If the creation is before the block of the hard fork, no subscription gets assigned to an Alias.
- Suppose the creation is after the hard fork. Then a subscription is created and assigned to the Alias and the creator account.
- If a subscription already exists, only the alias content gets updated.



### API support
The OpenAPI should support the new transactions on the node.


## Backwards Compatibility  
This proposal is a hard-forking change, thus breaking compatibility with old fully-validating nodes. It should only deploy the code change with widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
