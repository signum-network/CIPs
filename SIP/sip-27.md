---
sip:  27
title: Proof of Commitment (PoC+) consensus
description: Combines PoC with a kind of PoS
author:  jjos,
status: Final
type: Standard
category: Core
created: 2021-03-25
---
## Motivation

Proof of Capacity (PoC) is a sound and fair consensus algorithm, field tested for more than 6 years in the Signum network. Sound because it can use off-the-shelf equipment and is energy efficient. Fair because it has a very low entry barrier and shows a more linear scaling.

However, due to its nature, PoC - as it is currently implemented - can show security issues resembling the [nothing at stake problem](https://golden.com/wiki/Nothing-at-stake_problem).
Signum plot files can be easily reused by copycat PoC coins.
On one hand this reduces mining costs and could increase total network capacity. On the other hand, however, this can be seen as a weak point, since mining an additional coin with the same plot files also only adds the small cost of an additional sweep on the plot files.

## Abstract
The proposed concept is introduced and discussed in more detail in the following article [Proof of Commitment (PoC+): a Proof of Capacity Upgrade](https://jjos2372.medium.com/proof-of-commitment-poc-a-proof-of-capacity-upgrade-3131775e7a83).

In PoC+, the deadline is computed as:

`deadline = ln(hit/(commitmentFactor*baseTarget)) * 240 / ln(240)`

The difference between the current deadline formula and the equation above is the introduction of the `commitmentFactor`.
Miners with more capacity should have smaller `hit` values and by adding a factor to the miner's hit, we can account not only for the stored capacity the miner commits to the process (exactly as is today) but also for the amount of Signa committed to the mining process (similar to *stake*).


## Specification

Compute deadlines as follows:

`deadline = ln(hit/(commitmentFactor*baseTarget)) * 240 / ln(240)`

with the `commitmentFactor` limited to the interval [1/8, 8], given by the following formula:

`commitmentFactor = pow(commitmentRatio, 0.4515449935)`

where the `commitmentRatio` is given by:

`commitmentRatio = commitment / averageCommitment`

where `commitment` is the miner's committed amount in Signa per TiB and `averageCommitment` is the average value committed by the network.

The `averageCommitment` is given by a moving window average based on the past 24 blocks as follows:

`newAverageCommitment = (averageCommitment*23 + commitment)/24`

with `commitment` being the amount the miner forging the present block has committed in the process. There is also a limit of 20% change per block to reduce oscillations.

### Add and Remove Commitment

The miner `commitment` is given by the amount in Signa the user *commits* to mining.
This is accomplished by a new *Add Commitment* transaction (type=20, subtype=1).
This transaction *locks* a given amount and becomes effective after 60 confirmations.
After that period the miner can *unlock* it by using the *Remove Commitment* transaction (type=20, subtype=2).
In the case where a miner actually forges a block, his entire committed amount is locked for 1440 blocks.

### Miner Capacity Estimation

To keep the consensus fair and as linear as possible, the miner `commitment` is always given in Signa/TiB. The amount committed in Signa is easily accessible and the mining physical capacity is estimated as follows:

`estimatedCapacity = genesisTarget/1.83*nBlocksMined/(capacityBaseTarget * capacityEstimationBlocks)`

where `genesisTarget=18325193796` and `nBlocksMined` is the number of blocks mined *recently*.
The `capacityEstimationBlocks` is 360 if the miner has forged 3 or more blocks in the past 360 blocks and 32400 otherwise.

If the `estimatedCapacity` is smaller than 1 TiB, it is assumed to be 1 TiB.

### Effects on Mining and Mining Software
The proposed upgrade can be implemented with no effect on mining software or plot files.
In the proposed implementation, the deadline confirmation returns the *legacy* value to miners so they can be unaware of the upgrade and still function.

### Effects on Pool Software
Pool software needs to be upgraded to actually compute the fair share for miners, in case they commit Signa in the mining process. The [Signum Pool](https://github.com/signum-network/signum-pool) already has an open-source implementation with the necessary changes.

## Improved Robustness against 51% Attacks
So-called 51% attacks are a frequent concern for any distributed consensus. With the PoC+ hybrid consensus, a miner/pool would only be able to succeed in a 51% attack if they have 51% of the plotted disk space and at the same time have the average value in Signa available on their miner ID. If a miner has 51% of the disk space (or even much more than that) but zero Signa committed, the effective mining power would be reduced by up to 8 times. If a miner has a lot of Signa committed (staked) but low capacity, again a majority attack is impossible. 

If a miner has for instance 20% of the network in disk space and 100 times the average value committed, as soon as more blocks are mined, the average commitment could increase, thereby reducing the effective capacity of such a miner.

As can be seen, PoC+ not only increases the commitment in the mining process but also makes the consensus much harder to attack. A malicious actor cannot simply divert capacity currently mining other coins, but would also need to simultaneously stake a substantial coin amount for a non negligible period of time.

## References

* [Proof of Commitment (PoC+): a Proof of Capacity Upgrade](https://jjos2372.medium.com/proof-of-commitment-poc-a-proof-of-capacity-upgrade-3131775e7a83)
* [Signum Pool](https://github.com/jjos2372/babel-pool)
* [Nothing at stake problem](https://golden.com/wiki/Nothing-at-stake_problem)

## Backwards Compatibility
This is a hard forking change, thus breaks compatibility with old fully-validating node. It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
