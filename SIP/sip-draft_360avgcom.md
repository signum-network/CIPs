---
sip: <to be assigned>
title: Change Average Commitment Block Length
description: Increase the Network Average Commmitment Calculation from 24 blocks to 360 blocks
author: Bryan Bartlett (@Valareos)
status: Draft
type: Standard
category: Core
created: 2022-10-02
requires: SIP-27
---
  
## Abstract
This SIP seeks to change the average commitment calculation from `newAverageCommitment = (averageCommitment*23 + commitment)/24` to `newAverageCommitment = (averageCommitment*359 + commitment)/360` for better short term stability.

## Motivation
With the introduction of POC+, Commitment has become an intergral part of Signum.  Miners rely on a stable Average Network Commitment to predict mining revenue and ensure they are making the most of their hard drives. However, under the current system, there is a huge variance in the average commitment that can result in 80% swings in a matter of a few blocks.  This SIP strives to fix this by increasing the number of blocks the Network uses to calculate the average commitment.

## Specification
### Proposed changes
`newAverageCommitment = (averageCommitment*23 + commitment)/24` 
Should be changed to 
`newAverageCommitment = (averageCommitment*359 + commitment)/360`

This change will decrease volitility in short term average commitment, while still allowing long term movement as dictated by miner's commitment choices. More information can be found by reading [Simulation of Signum Network with Random Average Commitment](https://hive.blog/hive-146999/@btfg/simulation-of-signum-network-with-random-average-commitment)
  
### Effects on Mining and Mining Software
The proposed upgrade can be implemented with no effect on mining software or plot files.

### Effects on Pool Software
The proposed upgrade can be implemented with no effect on Pool Software. This change only affects the network's calculation of average commitment

## Rationale
360 Blocks was chosen as it would average the changes to the network average commitment over a full day. However, as show in in the Reference, there may be a smaller time period that also will suit the goals of this SIP, and should be discussed and analyzed.
  
## Backwards Compatibility
This is a hard forking change, thus breaks compatibility with old fully-validating node. It should not be deployed without widespread consensus.


## Reference Implementation
*[Simulation of Signum Network with Random Average Commitment](https://hive.blog/hive-146999/@btfg/simulation-of-signum-network-with-random-average-commitment)

## Security Considerations
This SIP only seeks to change a value in an already existing function. There are no addtional Security Considerations with this change as it does not introduce any vunerabilities or significant change to the core workings of the Signum Network. 

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
