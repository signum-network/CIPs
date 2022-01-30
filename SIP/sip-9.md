---
sip:  9
title:  To-All Transactions 
description: To-All Transactions
author: PoCC/rico666 (bots@cryptoguru.org)
status: Stagnant
type: Informational
created: 2018-07-16
---

## Abstract
To-All transactions are a way of interaction of a single individual (address) with the Signum community in its entirety (all addresses). A To-All transaction triggers a payment of a sum X that is being proportionally disseminated among all existing addresses with funds above a certain threshold.

> Lost coins only make everyone else's coins worth slightly
  more. Think of it as a donation to everyone.

*<div style="text-align: right">-- Satoshi Nakamoto</div>*

## Motivation

### Burning Signum 

There is no efficient way to burn Signa coins. One could send them to an address that is presumably without known private key (similar to [1BitcoinEaterAddressDontSendf59kuE](https://www.blockchain.com/en/btc/address/1BitcoinEaterAddressDontSendf59kuE)
in Bitcoin), but you never know if that private key really does not exist.

Another possibility is to send so small amounts to various addresses, that it is by orders of magnitude more expensive to retrieve that amount than its nominal value. Without any limits on "Dust" (see [SIP-8](sip-8.md)), it is conceivable to "burn Signa" by e.g. sending Multi-Out Same transactions with 1 Planck to random addresses.

The minimum cost of burning these Signa is the same as is minimum retrieval cost. E.g. currently a fully populated Multi-Out Same could burn 128 Planck at the cost of 735000 Planck.

### Giving Signa Away

The quote above from Satoshi Nakamoto illustrates that a "forgotten private key" is equivalent to *a donation to everyone*. So if an individual performs such a donation to everyone and effectively burn Signa for himself, we did achieve something equivalent to a "Proof of Burn", but without actually losing the coins from the total supply.

### PoS: The Other side of the coin

If we look at a To-All transaction being a "Proof-of-Burn" (PoB), then the complement on the recipient side is equivalent to Proof-of-Stake: Accounts with a Signa balance get their share from these PoB events proportional to their balance. In short, the more Signa you have, the
more you earn.

## Specification

Given an amount *M* of Signa to burn and the total supply being *N*.
The Dust threshold for accounts is computed as

    Balance > FEE_QUANT and
    Balance > N / (M * 100 000 000)

### Examples

Your account balance is FEE_QUANT or less: No matter how many Signa  are burnt, your account will receive no Plancks.

If the total supply (N) is 1980000000 and 1 (M) Signa is burnt, then the 1-Planck threshold is for accounts that have a balance of 19.8 Signa .

Because to-all transactions will adhere to dust threshold recommendations, in the case of burning one Signa, your lowest account balance in the case above is FEE_QUANT * 19.8 Signa = 14,553,000 Signa

In the case 5000 Signa are burned, the lowest account balance to receive Signa is 14,553,000 / 5,000 = 2910,60 Signa


### Dissemination Example

For simplicity, let's assume there are at this moment 5 accounts A₁, ..., A₅ in existence, where A₅ wants to send 5000 Signa "To-All" and the accounts are with the following balances:

    A₁:    400 Signa
    A₂:    700 Signa
    A₃: 15 000 Signa
    A₄:  8 000 Signa
    A₅: 11 000 Signa

Then we have a total of 400+700+15000+8000+11000 = 35100 Signain existence. However, for our computations, we need to take the 5000 Signa to be sent out of the total sum to remove their self-referentiality. This leaves us with the following percentages of the accounts:

    A₁%:   0.01328903654485049834   (400/30100)
    A₂%:   0.02325581395348837209   (700/30100)
    A₃%:   0.49833887043189368771 (15000/30100)
    A₄%:   0.26578073089700996678  (8000/30100)
    A₅%:   0.19933554817275747508  (6000/30100)
    -------------------------------------------
    total: 1.0                    (30100/30100)
    ===========================================

On the current MainNet, the number of accounts is around 70000 and the total Signa is the Signa mined so far ~ 1.96 bn Signa. The computational effort is much higher but the same principle applies.

So A₅ paying 5000 Signa To-All, will result in the following layout of transfers:

    A₁:   0.01328903654485049834 * 5000 =   66.44518272
    A₂:   0.02325581395348837209 * 5000 =  116.27906977
    A₃:   0.49833887043189368771 * 5000 = 2491.69435216
    A₄:   0.26578073089700996678 * 5000 = 1328.90365448
    A₅:   0.19933554817275747508 * 5000 =  996.67774086 + 1 Planck
                                          -------------
                                          4999.99999999 + 1 Planck


## Discussion

The current balance of Signa on the Poloniex and Bittrex exchange accounts at the time of this publication (2018-08-17, see also [Bittrex](https://explorer.signum.network/address/13383190289605706987), [Poloniex](https://explorer.signum.network/address/5810532812037266198)) is around 57% of the currently available supply.

This means that in case of a "To-All transaction", these exchange accounts together would get 57% of the burned Signa. At the moment, the exchanges most certainly have no process to propagate these PoS-like earned Signa to the individual account holders (i.e. pass them on).

The exchanges would be the biggest beneficiaries of To-all transactions, while the users with Signa on these exchages would be abstained from receiving the Signa they should, clearly there are two possibe consequences:

1) get the exchanges to have the customers participate (pass it on)
or
2) get your Signa in your private wallet

As 2) is detrimental for the exchanges, we believe an arrangement can be made, so exchanges implement 1)

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
