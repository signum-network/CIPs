---
sip: 38
title: Map support for smart contracts
description: Add the option to save/read arbitrary data inside contracts
author:  jjos
status: Review
Last Call: 2022-04-24
type: Standard
category: Core
created: 2022-04-17
---
# Abstract
Allow smart contracts to save and read arbitrary data in a (key1, key2)-value form.
Values saved are persistent and fully accessible.
This allows to write contracts that can interact with an unlimited number of participants.

Contracts can also read data from other contracts, allowing to easily implement *oracle* applications
or other type of communication between contracts without actually having to send transactions between them.

## Specification

The following new operations are included in the virtual machine:
 - `GET_MAP_VALUE_KEYS_IN_A`: get the value stored with keys in A1, A2, and A3 (if A3==0 use the AT ID as key3)
 - `SET_MAP_VALUE_KEYS_IN_A`: save the value in A4 with the keys in A1 and A2

## Backwards Compatibility  
This is a hard forking change, thus breaking compatibility with old fully-validating nodes.  
It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
