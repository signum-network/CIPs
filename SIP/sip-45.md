---
sip:  45
title: Staking Contracts
description: A standard for Staking Contrats
author:   frank_the_tank
status: Review
type: standard
category: SRC
created: 2022-11-17
---
# Introduction
Staking contracts allow participants to pledge a smart token with a specified lock-up period to earn Signa and/or another smart token distributed by the contract. The distribution of a staking contract can be fed by topping up the contract with the required funds or by a continuous income from other source. With  [SIP-30](sip-30.md)  the new standard can be simply used by linking the smart contract code from the first contract deployment of this SIP-45 standard. An audit check can be done by comparing the hash of the created Staking Contract with the hash from this original contract deployment- if both are the same the original code is used.

SRC-45 Contract deployment : https://chain.signum.network/at/7074088231427605580

Transaction: 7074088231427605580

Reference Hash: 4C840330C4352C62871D34CFBD0242F68F551FDBB9C12E013A1489A26009E16D

Code-Hash-ID:  15155055045342098571


# Motivation
The following standard allows the implementation of a standard API for Staking Contratcs as smart contracts. This standard provides basic functionality to setup and run Staking Contracts.

**What is a Staking Contract?**

A staking contract on the Signum chain is used to pay participants a reward through a fiduciary pool of specified smart token. Each deposit of the specified smart token into the staking contract will result in a 1:1 payout of a staking contract smart token in exchange.
Distributions will be made in favor of this smart token created by the staking contract.
A staking contract may have an expiration date or be active for infinity.

Signum deserves a proper standard for staking contracts which can be used in a fiduciary way, SIP-45 came to solve that.

We consider use cases for deployments that can be performed by individuals or corporations. Staking contract can be used to insentive holders or generate passicve income by delagte revenues to this contract. Examples:

-   Delagte royalities from NFT sales to a staking contract
-   Paying extra yield for long-term holders
-   Pooling smart tokens to by pass distributio to holder instrudactions with high position requirement

##  Java code for the SRC-45 contract
The Java code for the NFT standard looks as follows:

```java

        // Storing current total staking
        setMapValue(ONE, blockheight, totalstaked);
        // store the distribution of SIGNA if any
        if (distributedAmount > ZERO){
		    setMapValue(TWO, blockheight, distributedAmount);
        }
        // store distribution of distrubteToken if any
        if (distributedQuantity > ZERO){
            setMapValue(THREE, blockheight, distributedQuantity);
        }
    }
    private long distributionFee(long stakingholders){
        checkPlanckForDistribution = (stakingholders * DISTRIBUTION_FEE_PER_HOLDER)+ DISTRIBUTION_FEE_MINIMUM_HOLDER;
        if(checkPlanckForDistribution < DISTRIBUTION_FEE_MINIMUM){
            checkPlanckForDistribution = DISTRIBUTION_FEE_MINIMUM;
        }
        return (checkPlanckForDistribution);
    }

    private void checkRatio(){
        if(signaRatio > ZERO){
            maxAmountPerPayment = calcMultDiv(totalstaked, PLANCK_TO_SIGNA, digitsFactorPledgeToken) / signaRatio;
            if (maxAmountPerPayment < minAmountToDistributeSigna){
                maxAmountPerPayment = minAmountToDistributeSigna;
            }
        }
        if(tokenRatio > ZERO){
            maxQuantityPerPayment = (totalstaked/ digitsFactorPledgeToken) * digitsFactorYieldToken / tokenRatio;
            if (maxQuantityPerPayment < minQuantityToDistributeYieldToken){
                maxQuantityPerPayment = minQuantityToDistributeYieldToken;
            }
        }
    }
    private void distributeToStakingToken(){
        stakingholders = getAssetHoldersCount(qualifiedMinimumQuantity, stakingToken);
        if (stakingholders > ZERO){
            distributionFee = distributionFee(stakingholders);
            balanceCheck = this.getCurrentBalance() -distributionFee-CONTRACT_FEES ;
            if(balanceCheck  >= minAmountToDistributeSigna ){
                if (balanceCheck > maxAmountPerPayment && maxAmountPerPayment != ZERO){
                    distributedAmount = maxAmountPerPayment;
                }
                else{
                    distributedAmount = balanceCheck;               
                }
            }
            quantityCheck = this.getCurrentBalance(yieldToken);
            if ( quantityCheck >= minQuantityToDistributeYieldToken && yieldToken != ZERO ) {
                if( quantityCheck > maxQuantityPerPayment && maxQuantityPerPayment != ZERO){
                    distributedQuantity = maxQuantityPerPayment;
                }
                else{
                    distributedQuantity = quantityCheck;
                }
            }
            if(distributedAmount + distributedQuantity > ZERO){
                if(!distributionDone){
                    lastBlockDistributed =blockheight;
                    distributeToHolders(qualifiedMinimumQuantity, stakingToken, distributedAmount, yieldToken, distributedQuantity);                         
                }
            }   
        }
    }

    @Override
    public void txReceived() {
      // do nothing, since we are using a loop on blockStarted over all transactions
    }
```

### Creator of the Staking Contract
The creator is always the account which uploads the transaction for the creation of th staking contract to the chain.

### Variable setting (data stack)
While the creator is uploading the contract to the chain, the individual data stack for the staking contract should be defined within the transaction. Within the data stack the values should be set in the following order:

**stakingTokenTicker**
The  ticker of the generated staking token by the contractr with up to 10 numbers/letters.

**stakingTokenDecimals**
Number of decimals places for the staking token - should be the same as the token to stake.

**pledgeToken**
Id of the token which can be staked on the contract.

**digitsFactorPledgeToken**
Multiplyer for the ratio caluclation by digits of the **pledgetoken**
0 digits 1 , 1 digt = 10 ... 8 digits = 100000000

**yieldToken**
The Id of the token which should be distributed by default -if a postion exits on the contract.

**digitsFactorYieldToken**
Multiplyer for the ratio caluclation by digits of the **yieldToken**
0 digits 1 , 1 digt = 10 ... 8 digits = 100000000

**airdroppedTokenMinimumQuantity**
Minimum Quantity of a token to get distributed, which got airdroped to the contract.

Distribution parameter:

**paymentInterval**
Minimum number of blocks the contract is waiting until the next distribution will be executed.
If set 0 no interval is checked.

**contractExpiryInMinutes**
A value in minutes to calculate the exipry of the contract by creation.
If set to 0 the contract is infinity.

**qualifiedMinimumQuantity**
The minimum quantity needed of the stakingToken to be eligible for a distribution.

**minAmountToDistributeSigna**
The minimum amount of Signa required on the contract balance to trigger a payout.

**minQuantityToDistributeYieldToken**
The minimum quantity of the **yieldToken** required on the contract balance to trigger a payout.

**maxAmountPerPayment**
Maxium amount of Signa paid out at every **paymentInterval**.
If set to 0  the contract balance will be used to calculate the amount of Signa for a payout.

**maxQuantityPerPayment**
Maxium quantity of **yieldToken** paid out at every **paymentInterval**.

**signaRatio**
Besides a static **maxAmountPerPayment** a ratio can be defined how many StakingTokens are needed to get one Signa. Depending on the number of existing StakingTokens the **maxAmountPerPayment** is calculated and set for each **paymentInterval**.
This function is inactive when set to 0.
Example: 100 stakedToken should get 1 Signa  means 100:1. signaRatio = 100

**tokenRatio**
Besides a static **maxQuantityPerPayment** a ratio can be defined how many StakingTokens are needed to get one **yieldToken**. Depending on the number of existing StakingTokens the **maxQuantityPerPayment** is calculated and set for each **paymentInterval**.
This function is inactive when set to 0.
Example: 1000 stakedToken getting 0.2 yieldToken means 200 : 1. tokenRatio = 200

**lockPeriodInMinutes**
A value in minutes to calculate the lockout period for each **pledgeToken** by user address.
If this value is set to 0, no lockout period is calculated.

## Basic functions 
The contract will parse all transaction by the following basic logic:

#### Pledge Token check
The contract checks whether the transaction has transferred a **pledgeToken**.
If so, the same amount of **pledgeToken** is minted and sent to the sender of the transaction. If a **lockPeriodInMinutes** is set, a lock period for the address is stored in a map entry. Each time a new transfer is made from the same sender address, the lock period is overwritten with the latest calculation.

If the contact allready reached it expiry date - the **pledgeToken** will be just send back to the sender.

```java
            quantityCheck = tx.getAmount(pledgeToken);
            if(quantityCheck > ZERO){
                if(contractActiveCheck){
                    mintAsset(stakingToken,quantityCheck);
                    sendAmount(stakingToken,quantityCheck, tx.getSenderAddress());
                    totalstaked += quantityCheck;
                    if( lockPeriodInMinutes > ZERO){
                        setMapValue(FOUR,tx.getSenderAddress().getId(),lockPeriodTimeEnd.getValue());
                    }
                }
                else{
                    //Send back token as contract ended
                    sendAmount(pledgeToken,quantityCheck, tx.getSenderAddress());
                }
            }
```
#### Withdrawal staking Token check
The contract checks whether the transaction has transferred a **stakingToken**.
If this is the case, the contract checks whether the lockup period has expired, send back the same amount of **pledgeToken** and burns the received **stakingToken**.

If the contract has set a **lockPeriodInMinutes** and no map entry for the sender  address was found a new map entry will be set and the **stakingToken** will be send back to the sender.

```java
            quantityCheck = tx.getAmount(stakingToken);

            if(quantityCheck > ZERO){
                lockUpCheck=getMapValue(FOUR,tx.getSenderAddress().getId());
                if (lockUpCheck > ZERO){
                    if (getTimestamp(lockUpCheck).le(getBlockTimestamp())){
                        sendAmount(pledgeToken, quantityCheck, tx.getSenderAddress());
                        totalstaked -= quantityCheck;
                        // burn stakingToken
                        sendAmount(stakingToken, quantityCheck, getAddress(ZERO));
                    }
                    else{
                        //send back stakingtokens - lockup period not reached
                        sendAmount(stakingToken, quantityCheck, tx.getSenderAddress());
                    }
                }
                else{
                    //send back stakingtokens
                    sendAmount(stakingToken, quantityCheck, tx.getSenderAddress());
                    //regsiter address 
                    setMapValue(FOUR,tx.getSenderAddress().getId(),lockPeriodTimeEnd.getValue());
                }
            }
```
### Distribute airdropped tokens
Any address can request to distribute  airdropped tokens at any time.
The function checks if the amount needed for the distribution execution is included in the transaction and if the **airdroppedTokenMinimumQuantity** is met.
If both checks are valid, the airdropped token will be distributed over the **stakingToken** holders.

```java
            if(arguments.getValue1() == DISTRIBUTE_TOKEN_BALANCE){
                // only execute if token-id is not stakingToken or Token or distributeToken
                if (arguments.getValue2() != stakingToken && arguments.getValue2() !=pledgeToken && arguments.getValue2() !=yieldToken && arguments.getValue2() != ZERO){
                    //check balance of contract - only execute if above MimimumSize
                    if (this.getCurrentBalance(arguments.getValue2())>= airdroppedTokenMinimumQuantity && this.getCurrentBalance(arguments.getValue2()) > ZERO){
                        stakingholders = getAssetHoldersCount(qualifiedMinimumQuantity, stakingToken);
                        if(tx.getAmount() >= distributionFee(stakingholders)){
                            //distribute the token
                            if(!distributionDone){
                                distributeToHolders(qualifiedMinimumQuantity,stakingToken, ZERO,arguments.getValue2(), this.getCurrentBalance(arguments.getValue2()));
                                distributionDone =true;
                            }

                        }
                    }
                }
            }
  ```
### Cleanup
Only when the contract has expired and a final distribution has been made, the creator is entitled to remove the current balances of Signa and **yieldToken** from the staking contract.
```java
            if (arguments.getValue1() == CLEANUP_BY_CREATOR && stakingTimeoutLastPayment &&  tx.getSenderAddress().equals(getCreator())){
                sendAmount(yieldToken,getCurrentBalance(yieldToken),getCreator());
                sendBalance(getCreator());
            }
  ```


### Creation Costs
A creation of a stakingcontract will cost 2.5 Signa. To be paid with the deploy contract transaction.

### Activation costs
The contract has an activation cost of 0.9 Signa. Every transaction needs to have this minimum amount to interact with it, otherwise the transaction will not be handled within the smart contract. 

### Ensure payment interval
To ensure that each payment interval is executed even if there is no activity on the staking contract , we recommend that the contract creator set up a subscription with a payment of 0.9 Signa at an interval equal to the **paymentInterval**.

## Compatibility
This new staking standard is compatible with the current smart contract framework. 
**No** hard fork needed.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
