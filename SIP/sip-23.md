---
sip:  23
title: Enforce slot fees
description: Enforce slot fees
author:  CurbShifter
status: Final
type: Standard
category: Core
created: 2020-04-19
---
## Abstract
Block validation to strictly enforce the minimum fees used relative to the slot number in a block. This ensures fair distribution of fees among blocks and their miners.

## Specification

When validating a block the fee amounts of all transactions will be sorted in an ascending array. And checked to ensure these fees are at least 'FEE_QUANT' (735000) multiplied by the sorted array position. If this is not the case the block will be deemed invalid. This change also implies the removal of fat forging blocks where all fees are disregarded. Causing an unenforceable and unbalanced distribution of fees among the blocks and miners.

## Additional Notes
For reference, a previous less strict proposal checked the total amount paid of all transactions combined. This check implied that if someone overpays for a slot, another transaction could still have a lower fee and still be included in the block while the block stays valid. This would also have improved fair distribution of fees among the blocks and miners. However this favors dishonest miners that would use a privately held unconfirmed transaction (not propagated) which would only be self rewarded when a fat block is forged.


## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
