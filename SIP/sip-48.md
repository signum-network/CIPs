---
sip:  48
title: Alias renewal process
description: Introducing a new ownership handling for aliases on a yearly basis
author:   frank_the_tank
status: Review
type: standard
category: Core
created: 2022-12-10
---
## Abstract
This SIP will introduce temporary ownership of Aliases with an option to prolong it yearly.

## Motivation
Signum offers a simple setup to create aliases for an account. Once set/created, those aliases are owned by the creator account forever. With the introduction of [SIP-44](sip-44.md), the Alias will be more beneficial for domain, account or application pointers, making a reserved alias name more valuable. In addition, with [SIP-47](sip-44.md), a new service SNS (Signum Naming Service) will be introduced, a distributed, open, and extensible naming system based on the Signum blockchain. Given these cases, the creator/owner should actively maintain those Aliases to flag those aliases that are in use by them. Otherwise, another user should be able to claim the Alias.


## Specification
The following changes should be done with an upcoming hard fork to introduce the renewal process for aliases.

### Database changes
On the database level, the alias table will get a new field renewal_time which is an INT field and contains the timestamp until it stocks the Alias for the current owner. With the introduction of the needed hard fork, all aliases will get a DateTime of one year after the planned hard fork. In this case, all active users have enough time to check their current alias ownerships and can create a renewal subscription if desired.


### Node changes
**Transaction Set Alias**

If a user sends a setAlias transaction, it will execute the following DateTime handling for the Alias:
 - If the creation is before the block of the hard fork, it sets no DateTime on `renewal_time`.
 - If the creation is after the hard fork and no DateTime exits for the Alias, it sets a new DateTime with DateTime plus 24 hours on `renewal_time`. The same applies to a DateTime which is in the past or shorter than 24 hours in the future.
 - If the creation is after the hard fork, but the DateTime is longer than 24 hours in the future, no DateTime adjustment will happen on `renewal_time`.
 - If the setAlias leads to a new Alias, it sets the renewal_time with a value of 24 hours in the future.

In addition, if it executes the setAlias transaction for an existing Alias and the creator of the setAlias transaction is not the current owner, the transaction - which leads to an automated owner transfer - will only be executed if the renewal timestamp of the Alias has already expired.

## Handling of Alias offers(Sale)
If a `setAlias` transaction leads to an owner change, the node needs to check for ongoing Sales (public/private). If a sale exists, the node code removes the Sale, setting the `latest` field for this offer from 1 to 0.

**Auto renewal for aliases**

Signum already has all the basic functionality to empower the user for auto-renewal via the existing subscription transaction. A subscription is valid for the auto-renewal when it has the following parameters:

- Creating the subscription with an attachment. Setting a new field `alias` to bind the subscription to a given Alias.
- Paying the subscription amount to a defined receiver account.(Account Zero aka burning the fee)
- The subscription should have a minimum amount of 50 SIGNA as payment.
- Setting the subscription to a 1-year interval (in seconds).

**New node logic by execution of a subscription:**

The node will check the above parameters by creating an interval subscription payment. Suppose the creator of the subscription is still the owner of the Alias, and the account balance is sufficient to execute the subscription. In that case, the Alias will get an update on the renwal_time table field by one year and 24 hours (based on the execution time of the subscription). If the creator is no longer the alias owner, the subscription will cancel before any further execution.

**API support**

The current subscription transaction can create auto-renewal. Still, to make it more seamless for the user and developer, the following static parameters should introduce a new API call:

- **The receiver** is equal to the fixed receiver account
- **The frequency** is similar to one year in seconds
- **The amount** is equal to 50 Signa
- **The Alias** should be set with the aliasId to renew. Setting this attribute will create an attachment.

The transaction created with this call is still the same as the standard subscription creation.


###  Renewal costs
The proposal sets the renewal costs to 50 Signa. This amount is because a user needs to execute setAlias daily to update the reneval_time by every 24 hours. It would generate costs of 365 x 0.2 Signa = 73 Signa. By using the renewal with a subscription transaction, the set fee of 50 Signa leads to and discount of around 31% compared to a daily update.

### Multiverse support
To have later multiverse support for this feature, we should have a config entry for the receiver address to be a valid auto-renewal subscription.

## Backwards Compatibility  
This proposal is a hard-forking change, thus breaking compatibility with old fully-validating nodes. It should only deploy the code change with widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
