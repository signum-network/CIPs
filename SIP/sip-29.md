---
sip:  29
title: Mininum mining incentives
description: Mininum mining incentives
author:  frank_the_tank, jjos, shefas
status: Final
type: Standard
category: Core
requiers : SIP-27
created: 2021-03-25
---
## Introduction
There are many things that set Signum apart from other cryptocurrency platforms.
The first one is probably its innovative and sustainable consensus protocol. The original one is known as Proof of Capacity (PoC) and its evolution is known as [Proof of Commitment (PoC+)](sip-27.md). Or maybe because it was the first platform to run Turing complete smart contracts.
Another peculiar characteristic of Burstcoin is how block rewards are reduced - they decay by 5% every month. In this SIP the concept of a *minimum mining incentive* is proposed.

## Motivation
Miners secure the Signum network by reserving disk capacity (and optionally by committing a Signa balance in PoC+). Their incentive is the opportunity to make Signa when they forge blocks. Even currently, the amount of newly minted coins is substantially higher than the fees and is where miners make the most rewards. The block rewards keep miners financially invested in the chain's security, besides getting new coins into circulation. In most protocols, the block reward is set to decrease over time. Bitcoin is famous for its halvings, when the block reward halves (such as from 12.5 BTC to 6.25 BTC). These halvings happen every few years, and every time it happens, the network security would decrease if the price does not increase substantially. So reducing block rewards too quickly is actually a risky proposition.

In Signum, unlike Bitcoin, there are no halvings. Instead, block rewards are decreased by 5% every month, so the reduction is smoother but faster at first. Still, the open question for all cryptocurrencies is: "What will happen when the block reward hits zero?" The correct answer is very hard to predict.

Although Signum PoC+ can provide a very good level of security, even if the total network capacity is reduced, the larger the better. If the rewards reduce substantially, the only way to keep miners interested is by a unitary increase in coin price. But if the price increases too much, the network excludes the ones without sufficient monetary resources to transact in it.

An ideal solution to the zero block rewards scenario has yet to be discovered as no major mineable coin has entered this phase yet.  So there is no empirical data to delineate conclusions, only speculation. Bitcoin makes the bet that price increase will sustain the miners, even if it means excluding many use cases.

## Minimum Mining Incentive Proposal
Signum is much closer to having all the original supply mined and a different bet is proposed in this SIP. A bet that a very small minimum mining incentve of 100 Signa per block (amounting to less than 0.6% of the circulating supply in the first year and less than that in subsequent years) will be enough for future chain security and to allow miners to allocate resources with a long term plan.

If this is the perfect solution or the perfect balance between a very small inflation and long term security, only time will tell. The main criticism of these new incentives is a theoretically infinite total supply. This is actually a fallacy, since a fixed 100 Signa block reward introduces a decaying inflation (being 0.6% in the first year and decaying every year thereafter). Furthermore, the less than 0.6% inflation rate would likely be smaller than the amount usually lost or burnt in a given year, effectively making it deflationary overall. Finally, this minimum incentive is precisely known ahead of time, and can be accurately projected by investors, users, and miners in their decision making process.


## Specification

To use the following code to calculate the block rewards:
```java
  public static long getBlockReward(int height) {
	if (height == 0) {
	  return 0;
	}
	if (height >= 972_000) {
	  // Minimum incentive, lower than 0.6 % per year
	  return 100 * Constants.ONE_BURST;
	}
	int month = height / 10800;
	return BigInteger.valueOf(10000).multiply(BigInteger.valueOf(95).pow(month))
	  .divide(BigInteger.valueOf(100).pow(month)).longValue() * Constants.ONE_BURST;
  }
```
 
## Backwards Compatibility
This is a hard forking change, thus breaks compatibility with old fully-validating node. It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
