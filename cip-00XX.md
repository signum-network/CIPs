    CIP: ??
    Title: Updated AT fees
    Author: frank_the_tank, jjos, burstjack
    Comments-Summary: No comments yet.
    Comments-URI: 
    Status: Draft
    Type: Hard Fork
    Created: 2019-04-07


## Motivation

With Burst, smart contracts are implemented using Automated Transactions (AT), a technology created by the CIYAM developers. Automated Transactions are turing-complete and thus have a potentially infinite number of use cases. Beside this possible infinite number of use cases the current fee structure for AT on Burst-chain is counterproductive.
To attract business and developers for an investment/involvement with the ATs on the Burstcoin blockchain the fee structure needs to be reviewed and changed.


## Abstract

Any user with an BURST-Account is able to create an AT via the wallet or the corresponding API call. Unlike other blockchains, BURST does not have its own colored coin for running ATs.Registrating ATs on chain is supported by fees in BURST while execution of AT code is supported by the AT BURST balance.Initial setup of an AT costs BURST but the neck-breaker of the current implementation is the fee structure for keeping the AT alive. While in a recent hard-fork (CIP03) the new minimum fee of 0.00735 BURST was introduced (previous value was 1 BURST), the adaptation for the AT handling was forgotten or overlooked. Also the number of function calls an AT can do within a block was not adjusted which forces a complex AT to be dispensed over several blocks before the AT got finished.
Thus, in this proposal a new fee scheme for AT creation and execution is suggested.


## Specification

The parameters for the AT fee structure are handled in the AT_Constants.java file of Burst Reference Software (BRS). We propose to update the parameters in the following way (constants not changed are not listed):


| Constant | AT version1 | AT version 2|
|----------|-------------|-------------|
| STEP_FEE |  0.1 BURST  | FEE_QUANT/10|
| MAX_STEPS| 2000        | 100000      |
| COST_PER_PAGE| 1 BURST | 10*FEE_QUANT|

### Explanation of Parameters

**STEP_FEE** : This is the fee charged on every simple operation, like simple additions or subtractions.

**MAX_STEPS** : The maximum number of steps (operations, steps, API calls) to be included in a block. When this number is reached the current state is stored and the execution continues in the next block.

**COST_PER_PAGE** : This is the fee to register (or create) a new AT in BURST per page of the AT program, each page occupying 256 bytes. When an AT is created, this is accomplished by a special transaction which should contain at least COST_PER_PAGE times the number of pages required by the AT bytecode. Additionally, since the AT creation also is a transaction, it is also subject to the current slot fee mechanism.

### Effects of fee adjustment on AT deployment and runtime costs

In the following table typical registering costs as well as runtime ones are listed (all values are in Burst and numbers in parenthesis are the cost with the current AT fees):

![AT cases](./AT-cases.png)


Taking the assumption of one BURST at one USD as a possible future scenario, it is clear that the current AT fees can make most applications infeasible. In this scenario, a low complexity AT (see table above) would cost around 7 USD per run (i.e. per transaction received). A more complex case can cost around 2’000 USD per run. 


### Spam Attacks are still costy

The basic assumption is that the fee mechanism currently available is effective in preventing spam attacks exploiting standard transactions/messages/etc. Thus, AT Spam attacks need to be at least as costly as they would be if the attacker uses regular transactions.

1. STEP_FEE: was reduced to a FEE_QUANT/10 == 0.000735. This should be enough to avoid spam attacks by indefinitely running code. Since each simple addition, subtraction, etc cost as much as one tenth of the minimum fee of an entire transaction. Another possible attack is to program the following AT: it has a very large number of variables (costly to register), but runs very few steps (perhaps a single one) and is put to sleep until the next block. This is already prevented because every time the sleep function is called at least an API fee is already charged (10*STEP_FEE).

1. MAX_STEPS: increasing this number by a factor of 50  will only reduce the load on the system. Limiting the number of steps per block can only force to store intermediate AT variable states in the blockchain, which only take space without additional benefits for miners or nodes. Further, this limit can be considered redundant with the STEP_FEE. Since each step already has a fee, an AT programmer already has the incentive to keep the steps minimal. Finally, the system already has another protection, which put an AT in pause if the AT balance reaches zero. This already avoids any kind of infinite recursion.

1. COST_PER_PAGE: this cost is being reduced to a FEE_QUANT*10 == 0.0735 per page (256 bytes) when a new AT is registered (or created). Large AT programs can take several pages taking then more space in the blockchain and then affecting nodes. Even with the proposed modification each page (256 bytes) cost more than a message transaction can cost. Further, a message transaction can contain much more than 256 bytes (up to 4x times more). Under another perspective, registering a large (complex) AT mean potential for a large number of steps. This is, however, already taken care of by the step fees.


### Compatibility

The change will require a hard-fork. Any ATs created before the fork will be subject to the previous set of fees and will operate without any change. Only ATs created after the fork will run under the new fees.

## References

BRS version example with new  setup :https://github.com/jjos2372/burstcoin/commit/53ba6c6dea775371c017256529e16289e808257b

## Copyright

This document is placed in the public domain.
