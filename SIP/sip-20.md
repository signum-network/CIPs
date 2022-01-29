---
sip:  20
title: Updated AT fees
description: Updated AT fees
author: frank_the_tank, jjos, burstjack
status: Final
type: Standard
category: Core
requires : SIP-3
created: 2019-04-07
---
## Motivation

With Signum, smart contracts are implemented using Automated Transactions (AT), a technology created by the CIYAM developers. Automated Transactions are turing-complete and thus have a potentially infinite number of use cases. Beside this potentially infinite number of use cases the current fee structure for ATs on the Signum blockchain is counterproductive. To attract business and developers for an investment / involvement with the ATs on the Signum blockchain the fee structure needs to be reviewed and changed.

## Abstract

Any user with a Signum Account is able to create an AT via the wallet or the corresponding API call. Unlike other blockchains, Signum does not have its own colored coin for running ATs. Registering ATs on chain is supported by fees in Signa while execution of AT code is supported by the AT's balance. Initial setup of an AT costs Signa but the neck-breaker of the current implementation is the fee structure for keeping the AT alive. In a recent hard-fork (Pre-Dymaxion HF) the new minimum fee of 0.00735 Signa was introduced (as described in [SIP-3](sip-3.md), previous value was 1 Signa), however the adaptation for the AT fee structure was forgotten or overlooked. Also the number of API calls an AT can do within a block was not adjusted which forces a complex AT to be dispensed over several blocks before the AT is finished. Thus, in this proposal a new fee scheme for AT creation and execution is suggested.

## Specification

The parameters for the AT fee structure are handled in the `AT_Constants.java` file of the Signum-Node. We propose to update the parameters in the following way (constants not changed are not listed, new parameters are specified relative to `FEE_QUANT` as described in SIP-3):


| Constant | AT version 1 | AT version 2|
|----------|-------------|-------------|
| `STEP_FEE` | `0.1 Signa`  | `FEE_QUANT/10 Signa` |
| `MAX_STEPS` | `2000` | `100000` |
| `COST_PER_PAGE`| `1 Signa` | `FEE_QUANT*10 Signa` |
| `MAX_MACHINE_CODE_PAGES` | `10` | `20` |

### Explanation of Parameters

**STEP_FEE** : This is the fee charged on every simple operation, like simple additions or subtractions.

**MAX_STEPS** : The maximum number of steps (operations, steps, API calls) to be included in a block. When this number is reached the current state is stored and the execution continues in the next block.

**COST_PER_PAGE** : This is the fee to register (or create) a new AT in Signa per page of the AT program, each page occupying 256 bytes. When an AT is created, this is accomplished by a special transaction which should contain at least COST_PER_PAGE times the number of pages required by the AT bytecode. Additionally, since the AT creation also is a transaction, it is also subject to the current slot fee mechanism.

**MAX_MACHINE_CODE_PAGES** : Maximum compiled code (machine code) of a contract. With availability of [Signum-SmartJ](https://github.com/signum-network/signum-smartj), more complex contracts are manageable and the 10 pages limit for machine code is easily reached.

### Effects of fee adjustment on AT deployment and runtime costs

In the following table typical registering costs as well as runtime ones are listed (all values are in Signa and numbers in parenthesis are the cost with the current AT fees):

![AT cases](.assets/sip-20/AT-cases.png)


Making the assumption of `1 Signa = 1 USD` as a possible future scenario, it is clear that the current AT fees can make most applications infeasible. In this scenario, a low complexity AT (see table above) would cost around 7 USD per run (i.e. per transaction received). A more complex case can cost around 2000 USD per run.


### Spam Attacks are still costly

The basic assumption is that the fee mechanism currently available is effective in preventing spam attacks exploiting standard transactions, messages, etc. Thus, AT Spam attacks need to be at least as costly as they would be if the attacker uses regular transactions.

1. STEP_FEE: Reduced to `FEE_QUANT/10 == 0.000735 Signa`. This should be enough to avoid spam attacks by indefinitely running code. Since each simple addition, subtraction, etc. costs as much as one tenth of the minimum fee of an entire transaction. Another possible attack is to program the following AT: it has a very large number of variables (costly to register), but runs very few steps (perhaps a single one) and is put to sleep until the next block. This is already prevented because every time the sleep function is called at least one API fee is charged (10*STEP_FEE).

1. MAX_STEPS: Increased by a factor of 50, which will only reduce the load on the system. Limiting the number of steps per block can only force to store intermediate AT variable states in the blockchain, which only take space without additional benefits for miners or nodes. Furthermore, this limit can be considered redundant with the STEP_FEE. Since each step already has a fee, an AT programmer already has the incentive to keep the steps minimal. Finally, the system already has another protection, which put an AT in pause if the AT balance reaches zero. This already avoids any kind of infinite recursion.

1. COST_PER_PAGE: Reduced to `FEE_QUANT*10 == 0.0735 Signa` per page (256 bytes) when a new AT is created. Large AT programs can take several pages, taking more space in the blockchain and affecting nodes. Even with the proposed modification each page costs more than a message transaction can cost. Furthermore, a message transaction can contain much more than 256 bytes (up to 4x times more). From another perspective, registering a large (complex) AT brings potential for a large number of steps. This is, however, already taken care of by the step fees.

### Summary

Unlike other blockchains, Signa does not have its own colored coin for running ATs. Registering and running ATs is supported by fees and the AT balance in BURST. With the current fees for ATs (version 1) any substantial increase in Signa price can make AT applications infeasible. In this CIP a new fee table is proposed (version 2). The proposed change would take place as a hard-fork and possibly attract new developers and applications for Burstcoin.

## Compatibility
The change will require a hard-fork. Any ATs created before the fork will be subject to the previous set of fees and will operate without any change. Only ATs created after the fork will run under the new fees.

## Additional notes
Upon activation, this SIP will be implemented in Signum-Node and the changes to fee structure will take effect during the next hard fork.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
