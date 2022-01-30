---
sip:  30
title: Green Contracts
description: No need to duplicate contract codes in the chain
author:  frank_the_tank, jjos
status: Final
type: Standard
category: Core
created: 2021-09-18
---
## Introduction
There are many things that set Signum apart from other cryptocurrency platforms. For instance, it was the first platform to have Turing-complete smart contracts running on a public blockchain. In this CIP, the concept of *carbon contracts* is suggested as a solution to tackle scalability problems when deploying smart contracts.

Signum smart contracts are computer code that is executed by Signum nodes. These contracts, as any other accounts on the platform, have unique addresses, have balances, and can send and receive transactions. More importantly, these contracts are bound by the rules dictated by their code. Since their code is public and immutable, smart contracts can be used to create a multitude of trustless decentralized applications, like decentralized exchanges, auction systems, lotteries, games, collectibles, etc.

## Motivation
Anyone can write a smart contract and *deploy* it to the Signum blockchain. Once deployed, a unique address is assigned to the contract and anyone can interact with it. The deployment is accomplished by a transaction that has the contract code (its rules) attached. This transaction and its attachment have to be stored forever, which causes the scalability problem that many chains face today. Many platforms simply choose to increase the block size and reduce block times, allowing the deployment of very large numbers of contracts, as long as fees are paid. This increases the processing costs for running a node. The result is final users being forced to use centralized services to interact with the chain, not being able to run nodes themselves. This trend jeopardizes decentralization and can make these chains worthless in the long run (*if you can't run a full-node yourself, then it's just another bank*).

Most contracts deployed are simply copies of existing contracts. They are new instances of the same code. A typical example is NFTs (non-fungible tokens), which can be deployed by the thousands, all with the same code (template). In this CIP, we suggest circumventing this problem with the concept of carbon contracts. **It consists of a contract deployment method where no duplicate code actually needs to be sent or stored**. You can deploy a contract by *carbon copying* an existing contract. By doing that, you use the same code, with the same security and transparency, that is already stored on the blockchain. As a result, the deployment of a *carbon contract* becomes trivial in terms of space taken in the chain or in the blocks. Further, the deployment of a carbon contract is independent of the original contract code size. This also means less fees for deployment.

## Specification

The behavior of the `createATProgram` API endpoint will be modified as follows:
 - Accept smart contract deployments with the `code` parameter empty.
 - If the `code` parameter is empty, the `referencedTransactionFullHash` parameter must point to a past contract deployment transaction.
 - The reference transaction is used to define the contract code and related parameters (`dpages`, `cspages`, `uspages`, and `minActivationAmountNQT`)
 
Once deployed, the carbon contract will behave exactly as it would if deployed with the actual code. The `getAT` and `getATDetails` endpoints will return the original contract code (will not be empty).
 
## Backwards Compatibility
This is a hard forking change, thus breaks compatibility with old fully-validating node. It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
