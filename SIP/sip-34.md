---
sip: 34
title: Round minimum fee to 2 digits
description: Makes the fee calculation more human friendly
author:  frank_the_tank, jjos
status: Final
Last Call: 2022-02-18
type: Standard
category: Core
requires: SIP-31
created: 2022-02-01
---

## Abstract
This SIP will adjust the minimum fee to 0.01 Signa from 0.00735 Signa.

## Motivation
Signum currently has a minimum fee of 0.00735 Signa which is the basis for calculating the cost per 176bytes per transaction see [SIP-31](sip-31.md) **New fee framework** and  is used as a multiplicator for some transaction types.

With this SIP we like to simplify the minimum fee to 0.01 Signa to make it simpler to use/remember and also make it much easier to calculate (for humans) the costs of a transaction.

## Specification
The FEE_QUANT must be set to 0.01 SIGNA
The Transactions will be multiples of the FEE_QUANT, defined by SIP-31.
The defined factors for Alias and Token-Creation in SIP-31 will be kept.
The current parameters for SmartContracts (StepFee, Cost_per_page) see [SIP-20](sip-20.md) will not be touched.

## Backwards Compatibility
This is a hard forking change, thus breaking compatibility with old fully-validating nodes. It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
