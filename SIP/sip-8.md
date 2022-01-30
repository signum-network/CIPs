---
sip:  8
title: Define Tx and Balance "Dust"
description: Define Tx and Balance "Dust"
author: PoCC/rico666
status: Stagnant
type: Informational
created: 2018-07-16
---

## Abstract
Signum is divisible by 8 digits, so the smallest unit of account for Signum- called a Planck - is 1/100 000 000th of a Signa . It is certainly possible - right now - to send 1 Planck to any address.

The current minimum FEE_QUANT (the smallest fee you can use to make a transaction) is 735 000 Planck. It is conceivable that in future where Signum has a much higher value, this fee might be lowered.

There are a few applications where sending 1 Planck seems reasonable. One could "burn Signa" by sending 1 Planck to random addresses via Multi-Out Same. Retrieving these would always be non-economical.

Currently such a transaction would send 128 Planck and cost at least 735 000 Planck. Only the 128 Planck could be considered "burnt", while the FEE_QUANT would go straight to miner reward and therefore remain in the economic cycle of Signum.

Addresses containing a balance less than FEE_QUANT should be considered "Balance Dust".

A payment transaction of any type (as of now Ordinary Payments, Multi-Outs and Subscriptions) should be considered "Transaction Dust" if the result of this transaction would lead to a target address containing "Balance Dust".

## Motivation
Depending on the context, but effectively for every type of transaction, there is a threshold below which the sums are economically irrelevant. This does result in rounding to the Nth digit, truncating or similar approximations.

As of today, FEE_QUANT is around 0.00735 US-cent and even with a Signum reaching parity with the USD, FEE_QUANT would still be below 1 US-cent. This threshold is for the foreseeable future low enough to not cause economic damage when rounding to it or considering only amounts higher than this.

Coupling the informal term "dust" to FEE_QUANT, protects small balance accounts even in the case Signum should rise significantly in value. FEE_QUANT would have to be lowered then to keep transaction cost economically feasible.

## Specification
As this SIP is an informal proposal, the Signum protocol will further _allow_ to make transactions whose economic impact is below FEE_QUANT. However, if this SIP is accepted, it shall not be considered a bug if a software or process does not.

The following rules shall apply on-chain:

1.  avoiding a transaction to move amounts smaller than FEE_QUANT is considered good practice
    
2.  the recipient of a transaction as defined in 1) has no authority to demand such a transaction
    
3.  avoiding a transaction that would result in a balance on the target adress smaller than FEE_QUANT is considered good practice
    
4.  the recipient of a transaction as defined in 3) has no authority to demand such a transaction
    

Limiting these rules on-chain means that any amount below FEE_QUANT in internal accounting systems (pending pool fees, exchange fees, etc.) may still be perfectly relevant.

This SIP does not forbid any transaction to happen. It merely defines which transactions are sufficiently "economically irrelevant", so the sender can decide to not perform them - and the receiver has to respect this decision.

### Examples

Sender A could send recipient B a total of 0.001 Signa (asset dividends, pool payout, ...), but won't as it is below FEE_QUANT. B has no right to demand this sum to be sent to him.

Recipient B has 0.005 Signa on his address. Sender A could send recipient B a total of 0.001 Signa , but won't because the resulting balance (0.006 Signa ) is smaller than FEE_QUANT. Recipient B has no right to demand this sum to be sent to him.transaction to happen.

Person C has a Signum address with the balance of FEE_QUANT + 1 Planck. It is perfectly reasonable for that person to issue a transaction costing FEE_QUANT tx fee to "save" that 1 Planck to some other address of theirs. As C is in the role of sender (and maybe even receiver), it's upon them to decide if this transaction shall happen or not.

## Discussion
This informal SIP defines a boundary of economic relevance. This is not a novel concept. BTC also knows the notion of "Dust" and for any rational participant in the Signum transaction network, moving funds that are smaller than the minimum tx fee necessary to move them is out of the question.

Neither does this SIP nor will the wallet forbid anyone to perform economically unfeasible transactions. You - as a sender - will always have the right to decide to make these transactions. Contrary to BTC, where a transaction with an output value lower than 5460 satoshi gets rejected with the message "TX rejected (code -22)", this is a much more liberal aproach.

Recipients are also protected by the binding of the term dust to FEE_QUANT. In case Signum should raise in value significantly, FEE_QUANT has to be lowered, as transaction cost would become exorbitant (With Signa at $100, the current FEE_QUANT would be $0.735)

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
