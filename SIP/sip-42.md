---
sip: 42
title: Subscription behaviour enhancement
description: Keep a subscription alive until it is cancelled by the user
author:  frank_the_tank
status: Review
Last Call: 2022-11-19
type: Standard
category: Core
created: 2022-10-09
---
## Abstract
Allow a subscription to persist even if the account balance is insufficient to pay at the interval check.

## Motivation
Subscriptions are not new to the Signum blockchain. They could be used since the start in 2014. The current implementation is similar to a standing order, which gets executed at each interval as long as the account balance allows it. If the account has irregular inflows of Signa it could happen that the subscription get cancelled by the blockchain as the balance was not sufficient. The user should get the possiblity to define if a subscription should be cancelled or just set on hold until the next interval will occure or the subscription gets really cancelled by the user. 

## Specification
The subscription should get a new attribute `KeepAlive`. This value is by default `false`. 
If set to `true` the subscription will continue even when the account-balance is not enough to pay by the given interval.

The node would skip the payment in this case, but add a new record on the DB with the next payment-interval, amount and latest = 1.

## Backwards Compatibility  
This is a hard forking change, thus breaking compatibility with old fully-validating nodes.  
It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
