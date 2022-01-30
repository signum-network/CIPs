---
sip:  13
title: Suggested Transaction Fees
description: Suggested Transaction Fees
author: PoCC/Brabantian
status: Final
type: Standard
categroy: Interface
requires: SIP-13, SIP-16
created: 2018-08-07
---
## Abstract
A number of transaction fee suggestions should be supplied to the user when they wish to do a transaction on the Signum network. The amount of these fees will depend on the recent usage of the Signum network. These suggestions try to balance cost and speed in different ways, enabling the user to pick one depending on the situation and his requirements.

## Motivation
When creating a new unconfirmed transaction, a variable fee can be supplied, as described in [SIP-3](sip-3.md). The higher the slot the unconfirmed transaction gets into, the higher the odds are that it will be picked up in the next block. Unconfirmed transactions with a very low fee have a higher probability to be delayed or to be dropped from the mempool entirely, if the chance of them being applied to the blockchain within their deadline becomes 0.

This means that depending on the higher importance of an unconfirmed transaction, a higher fee should be applied to it.  Due to the variable "transactional load" on the network, it is not possible to give a truly optimal fee once and for all. A too low fee might result in the unconfirmed transaction being applied many blocks away, a too high fee might give the feeling of overly expensive transactions.

The goal of the Suggested Transaction Fees, is to provide the user with a couple of different transaction fees that try to balance cost and speed. This will enable the user to pick a transaction fee suitable to his situation that keeps in mind the current "traffic" on the network.

## Specification

### Wallet Internals

The FeeSuggestion type has three values in it. One for each fee suggestion type. They balance between cost and chance of being added in the following block. The cheapFee fee is the cheapest one, and least likely to be added in the following block, priorityFee is the most expensive with the highest odds to be added.

```java
public class FeeSuggestion {

  private final long cheapFee;
  private final long standardFee;
  private final long priorityFee;
  /* ... */
}
```

The FeeSuggestionCalculator is the component that will calculate the fee suggestions. It keeps a circular buffer of a number of blocks that will be used to run the fee suggestion algorithm on. This buffer is of the type CircularFifoBuffer with a given size of 10. When a new block is added
to the buffer, the oldest one in the buffer will be removed. This means that any point, a maximum of 10 blocks is held in it.

Because these suggestions will always give the same result for the same blocks, the fee suggestion should be calculated once per block range. Once a new block is processed by the system, a recalculation will be done for new suggestions. This enables users to do different calls to the method, that will give a cached result.

In case the FeeSuggestionCalculator's latestBlocks is empty, an initial filling of the latest historical blocks will be done. Afterwards, additional blocks will be added through the AFTER_BLOCK_APPLY event.

```java
private final CircularFifoBuffer latestBlocks;

private FeeSuggestion feeSuggestion;

public FeeSuggestionCalculator(/* ... */ int historyLength) {
  this.latestBlocks = new CircularFifoBuffer(historyLength);

  /* ... */

  blockchainProcessor.addListener(block -> newBlockApplied(block), Event.AFTER_BLOCK_APPLY);
}

private void newBlockApplied(Block block) {
  if (latestBlocks.isEmpty()) {
    fillInitialHistory();
  }

  this.latestBlocks.add(block);
  recalculateSuggestion();
}

private void fillInitialHistory() {
  blockchainStore.getLatestBlocks(latestBlocks.maxSize()).forEachRemaining(latestBlocks::add);
}
```

To calculate the fee suggestions, multiple algorithms can be used. For the time being, a rather simple one will be used, but this might change in the future. The algorithm will base the fee suggestions on all the blocks currently in the latestBlocks buffer.

```java
private void recalculateSuggestion() {
  int lowestAmountTransactionsNearHistory = latestBlocks.stream().mapToInt(b -> ((Block) b).getTransactions().size()).min().orElse(1);
  int averageAmountTransactionsNearHistory = (int) Math.ceil(latestBlocks.stream().mapToInt(b -> ((Block) b).getTransactions().size()).average().getAsDouble());
  int highestAmountTransactionsNearHistory = latestBlocks.stream().mapToInt(b -> ((Block) b).getTransactions().size()).max().orElse(1);

  long cheapFee = (1 + lowestAmountTransactionsNearHistory) * FEE_QUANT;
  long standardFee = (1 + averageAmountTransactionsNearHistory) * FEE_QUANT;
  long priorityFee = (1 + highestAmountTransactionsNearHistory) * FEE_QUANT;

  feeSuggestion = new FeeSuggestion(cheapFee, standardFee, priorityFee);
}
```

### Probabilities of inclusion

Given the implementation details above, the following approximate probabilities hold true:

**cheap**: Chance of 50% to be included within the next 10 blocks
**standard**: chance of 50% to be included within the next block
**priority**: chance of 90% to be included within the next block, 99% within the next 2 blocks

### HTTP layer

A new "SuggestFee" operation is added to the HTTP layer, to be used by the Signum-Node  frontend - or other systems - to obtain the latest fee suggestions.

This operation will take no parameters, and return the three suggestion values in the "cheap", "standard", and "priority", fields.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
