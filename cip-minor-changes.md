    CIP: TBD
    Title: Minor Network Changes
    Author: harry1453
    Comments-Summary: Minor network changes to be carried out during Sodium Hard Fork
    Status: Active
    Type: Consensus (Hard Fork)
    Created: 2020-04-25

## Abstract

This document defines the small network changes that will be activated as part of the Sodium Hard Fork.

## Specification

### AT VM - MD5ATOB

Expected behaviour: The MD5 sum of registers A1 and A2 is copied into B1 and B2.

Actual behaviour: Bytes 9-16 of the MD5 sum of registers A1 and A2 are copied into the register B1, and B2 is left untouched.

This change rectifies this behaviour to match the expected behaviour.

### AT VM - getRandomIdForTxInA

Expected behaviour: The first 8 bytes of the sha256 hash of the previous block's generation signature followed by the transaction ID in register A1 is returned.

Actual behaviour: The first 8 bytes of the sha256 hash of the previous block's generation signature, followed by the transaction ID in register A1, followed by 56 zeroed bytes is returned.

This change rectifies this behaviour to match the expected behaviour.

### AT VM - putLastBlockGenerationSignatureInA

Expected behaviour: The generation signature of the previous block is copied into registers A1-A4.

Actual behaviour: BufferUnderflowException is thrown.

This change rectifies this behaviour to match the expected behaviour.

### Subscription fees

CIP 3 describes the new fee model. In the POCC's Pre Dymaxion HF, the minimum fee was lowered to 0.00735 Burst, but the fees for subscriptions still remained at 1 burst per subscription payment. This change makes the fee for a subscription payment 0.00735 Burst.

### Disabling effective balance leasing

Effective balance leasing is a feature that was in NXT to help with the PoS mining protocol. It is a dormant feature in Burst that can only be used by hand-crafting a transaction. This change disables it as it is not an intentional feature, and an NXT relic.

## Compatibility

This is a consensus breaking change, so will be activated during the Sodium Hard Fork.

## References

* [CIP 3](cip-0003.md)

## Copyright

This document is placed in the public domain.
