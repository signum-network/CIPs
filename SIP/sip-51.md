---
sip: 51
title: Lending Commitment
description: New lending functionality to share commitment
author: frank_the_tank
status: Draft
type: standard
category: Core
created: 2024-11-16
---
### Abstract
This SIP introduces a lending setup for commitment within the Signa ecosystem. The new concept is integrated into the core code of the Signum Node and is an extension to the existing POC+.

### Motivation
With the introduction of PoC+, miners can boost their mining output by adding commitment, which is stored on the miner's account and is only valid for its plots. Currently, Signa balances from other accounts cannot be used in mining without setting up an independent miner for those accounts. This SIP should enable Signa holders to lend commitment to miners safely and in a protocol-integrated way.

### **Proposed Transaction Types**

- **Offer Commitment** <br>
A new transaction type (type 20, subtype 3) where lenders specify the amount of Signa offered, the lending cost for a 1440-block period (4 days) and the minimum duration of the lending before it can be cancelled from the lender. The minimum duration number in blocks must be 0 or higher. Offers can be public or directed to a specific account, with a minimum rate of 1 Planck per period. Zero rates are not allowed. Transaction fee: 1 Signa. <br>
**Important note:** The Signa is never leaving the Lender account; instead, it gets internally moved from the **unconfirmed balance** to the **balance** of its account.

- **Accept Commitment Offer** <br>
A transaction type (type 20, subtype 4) allows an account to accept an Offer Commitment. The specified account or any other account can accept the commitment offer. The transaction fee equals the rate set by the Offer Commitment, but a minimum of 1 Signa.

- **Cancel Commitment Offer** <br>
A transaction type (type 20, subtype 5) enables the creator to cancel an Offer Commitment. If an offer has not been accepted, it can be cancelled at any time. If accepted, cancellation occurs not before the given block height reaches the minimum duration set by the lender by the transaction "Offer Commitment". If the minimum duration is reached, the cancel will be executed with the next subscription payment interval. Only the creator can execute the cancellation. Transaction fee: 0.1 Signa.

### **Database Structure for Commitment** <br>
A new Commitment table in the database needs to be created, and  will contain the following fields:

- **dbid**: Auto-increment identifier.
- **id**: Creator of the offer.
- **recipientid**: Set if the offer is accepted, marking the account that borrowed the Signa as Commitment.
- **commitment**: Amount of Signa offered for Commitment
- **rate**: Rate per 1440 blocks in Planck (min 1)
- **height**: Block height at offer creation.
- **from_height**: Block height when the commitment activates (Accept block + 1440 blocks).
- **min_duration_to_block**: Block height when cancel of a lender kicks in ( Accept block + number of min duration blocks)
- **isCanceled**: True if Cancel Commitment is executed.
- **latest**: 1 if active, 0 if removed.

### Lending Commitment Process

**Offer Commitment** <br>
The transaction is created if the creator has enough unconfirmed Signa balance to cover the transaction fee and the Signa to lend as Commitment. After being added to a block, it can be accepted by any eligible account or cancelled immediately by the creator.

**Activate Commitment** <br>
When an offer is accepted:

- **recipientid** is set in the Commitment table.
- **from_height** is updated in the Commitment table.
- **min_duration_to_block** is updated in the Commitment table 
- **A new subscription is created**, which automatically pays the rate every 1440 blocks to the offer creator (lender). 

As the first payment of the subscription is set to the activation of the Commitment, the recipientid always pays the rate upfront for the interval. This commitment is then used in mining calculations for the recipientid from the specified from_height onward. Miners may accept multiple Commitment Offers. 

### Limitation of factor boost
Instead of adding every borrowed commitment into the commitment factor calculator, the node checks the difference between the commitment factor only with the own commitment of the miner and the commitment factor resulting from the miner's commitment and the borrowed commitment. The factor improvement is capped to an addition of 2. This means a miner with no commitment can scale from 0.125 to 2.125 in maximum, and a miner with a factor of 4.533 can scale the maximum to 6.533 with borrowed commitment. <br>
Overall the maximum factor of 8 is given.

### **Commitment Cancellation Conditions**

- **Insufficient Balance** <br> 
If the recipient (miner) lacks funds for the subscription payment, the commitment and its renewal subscription will be cancelled, releasing the committed balance to the lender's unconfirmed Signa balance.

- **Subscription Cancellation** <br>
If the recipient (miner) cancels the subscription, the commitment is cancelled, and the balance is immediately returned to the lender's unconfirmed Signa balance.

- **Offer Creator Cancellation** <br>
When the creator sets a commitment to cancel, it is set to the next subscription payment interval after the minimum duration block height. By this, the commitment and renewal subscription is cancelled, and the balance is released to the lender’s unconfirmed Signa balance. Setting the field latest to 0 in the Commitment table.


### **Remarks**
With SIP-32, the activation delay for commitments was set to 1440 blocks. This SIP aligns with that standard, activating borrowed commitments for the recipient after 1440 blocks, and similarly restricts removal to 1440-block intervals. This standardisation ensures predictable commitment handling across all accounts.

### API support
The OpenAPI should support the new transactions on the node.

## Backwards Compatibility  
This proposal is a hard-forking change, thus breaking compatibility with old fully-validating nodes. It should only deploy the code change with widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

### Glossary
**total balance** =  The total account balance of an account is conducted based on an _unconfirmed balance_ and _balance_. <br>
**unconfirmed balance**  = The unconfirmed balance is free to use Signa on the account. <br>
**balance** =  The balance is Signa's blocked balance on the account. Those balances are used for Commitment, Trades(Buy), etc..
