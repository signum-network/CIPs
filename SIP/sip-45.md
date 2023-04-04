---
sip:  45
title: Staking Contracts
description: A standard for Staking Contracts
author:   frank_the_tank
status: Review
type: standard
category: SRC
created: 2022-11-17
updated: 2023-03-22
---
# Introduction
Staking contracts allow participants to pledge a smart token with a specified lock-up period to earn Signa and/or another smart token distributed by the contract. The distribution of a staking contract can be fed by topping up the contract with the required funds or by a continuous income from another source. With [SIP-30](sip-30.md), the new standard can be simply used by linking the smart contract code from the first contract deployment of this SIP-45 standard. An audit check can be done by comparing the hash of the created Staking Contract with the hash from this original contract deployment- if both are the same, the original code is used.

SRC-45 Contract deployment: https://chain.signum.network/tx/12825029980485714879

Transaction: 12825029980485714879

Reference Hash: bf03963cd1a8fbb11b90bb0fcba1dbd6384611df3d07bc0e4eca2fa5b9e50462

Code-Hash-ID:  3735293229655148902


# Motivation
The following standard allows the implementation of a standard API for Staking Contracts as smart contracts. This standard provides basic functionality to set up and run Staking Contracts.

**What is a Staking Contract?**

A staking contract on the Signum chain is used to pay participants a reward through a fiduciary pool of specified smart token. Each deposit of the specified smart token into the staking contract will result in a 1:1 pay-out of a staking smart token (minted by the contract) in exchange. Distributions will be made in favour of this smart token created by the staking contract. A staking contract may have an expiration date or be active for infinity.

Signum deserves a proper standard for staking contracts which can be used in a fiduciary way. SIP-45 came to solve that.

We consider use cases for deployments that can be performed by individuals or corporations. A staking contract can be used to incentive holders or generate passive income by delegating revenues to this contract.
Examples:

-   Delegate royalties from NFT sales to a staking contract
-   Paying extra yield for long-term holders
-   Pooling smart tokens to bypass distribution to holder instructions with high position requirements

##  Java code for the SRC-45 contract
The Java code for the NFT standard looks as follows:

```java
package bt.dapps;
import bt.Address;
import bt.BT;
import bt.Contract;
import bt.Emulator;
import bt.Register;
import bt.Timestamp;
import bt.Transaction;
import bt.ui.EmulatorWindow;

/**
 * A staking contract for the Signum Blockchain
 * 
 * The creator defines which token can be staked on this contract aka pledgeToken
 * The contract will mint 1:1 for each pledgeToken a stakingToken and send it to the address that submitted the pledgeToken.
 * 
 * Any income of Signa to this contract will be distributed to the stakingToken holders.
 * Also, another token aka yieldToken can be automated distributed to the stakingToken holders.
 * The contract can have an end-date; after a final payment nothing more is paid out.
 *
 * In the contract the following can be defined:
 * 
 * paymentInterval = Minimum blocks between distribution; 0 = no waiting
 * minAmountToDistributeSigna = Minimum Amount of Signa needed on the contract to trigger a distribution
 * maxAmountPerPayment = Maximum Amount of Signa which will be distributed when distribution is triggered
 * qualifiedMinimumQuantity = Minimum number of stakingToken needed to be eligible for the distribution
 * contractExpiryInMinutes = Staking end-time in minutes ( 0 = infinite)
 * yieldToken = Token to distribute if set
 * minQuantityToDistributeYieldToken = Minimum quantity of yieldToken needed before distributed per paymentInterval
 * maxQuantityPerPayment = Maximum quantity of yieldToken which will be distributed per paymentInterval
 * signaRatio = Ratio for the distribution = stakingToken:1  for Signa
 * tokenRatio = Ratio for the distribution = stakingToken:1  for yieldToken 
 * If signaRatio = 0 maxAmountPerPayment is used as static value otherwise pledgeToken on contract balance will define the maximum
 * If tokenRatio = 0 maxQuantityPerPayment is used as static value otherwise pledgeToken on contract balance  will define the maximum
 * 
 * lockPeriodInMinutes = Time of lock period in minutes for every token transfer send to the contract ( 0 = no lockup period set)
 * If contractExpiryInMinutes is set, the lockPeriodTimeEnd will be at most equal to the time of contractExpiryInMinutes calculated as datetime. 
 * 
 * @author frank_the_tank
 */
public class StakingDynamicContract extends Contract {
    // stakingToken parameter
    long stakingTokenTicker;
    long stakingTokenDecimals;
    // Decimals should be the same as from the token

    // Token to pledge
    long pledgeToken;
    long digitsFactorPledgeToken;

    // yieldToken to distribute by default
    long yieldToken;
    long digitsFactorYieldToken;
    // digit 0 = 1 ; digit 1 = 10 ... digit 8 = 100000000

    // Minimum Quantity for any other token (without digit adjustment)
    long airdroppedTokenMinimumQuantity;

    // Distribution parameter
    long paymentInterval;
    int contractExpiryInMinutes;
    long qualifiedMinimumQuantity;
    long minAmountToDistributeSigna;
    long minQuantityToDistributeYieldToken;

    // Distribution parameter for fixed payouts
    long maxAmountPerPayment; 
    long maxQuantityPerPayment;

    // Distribution parameter for dynamic payouts
    long signaRatio; 
    // Example: 100 stakedToken getting 1 SIGNA = 100:1 = 100
    long tokenRatio; 
    // Example: 1000 stakedToken getting 0.2 distribute Token = 200 : 1 = 200

     // lockPeriod - If set user can´t convert stakingToken into token
    int lockPeriodInMinutes;
    Timestamp lockPeriodTimeEnd;

   // stakingToken created by contract
    long stakingToken;

    // temporary variables
    Timestamp stakingTimeout; 
    boolean stakingTimeoutLastPayment;
    Timestamp lastProcessedTx;
    Register arguments;
    long totalstaked ;
    long stakingholders;
    long distributedAmount;
    long distributedQuantity;
    Transaction tx;
    long lastBlockDistributed;
    long checkPlanckForDistribution;
    long distributionFee;
    long blockheight;
    long quantityCheck;
    long balanceCheck;
    long lockUpCheck;
    boolean contractActiveCheck;
    boolean distributionDone;
    public static final long ZERO = 0;
    public static final long ONE = 1;
    public static final long TWO = 2;
    public static final long THREE = 3;
    public static final long FOUR = 4;
    public static final long DISTRIBUTE_TOKEN_BALANCE = 99;
    public static final long CLEANUP_BY_CREATOR =100 ;    
    public static final long DISTRIBUTION_FEE_PER_HOLDER = 100000;
    public static final long DISTRIBUTION_FEE_MINIMUM_HOLDER = 1000000;
    public static final long DISTRIBUTION_FEE_MINIMUM = 2000000;
    public static final long PLANCK_TO_SIGNA = 100000000;
    public static final long CONTRACT_FEES = 90000000;
    public StakingDynamicContract() {
	    // constructor, runs when the first TX arrives
        stakingToken = issueAsset(stakingTokenTicker, 0L, stakingTokenDecimals);
        if ( contractExpiryInMinutes > ZERO){
            stakingTimeout= getBlockTimestamp().addMinutes(contractExpiryInMinutes);
        }
        if(lastBlockDistributed == ZERO){
            lastBlockDistributed = this.getBlockHeight();
        }
        if (lockPeriodInMinutes > ZERO){
            lockPeriodTimeEnd = getBlockTimestamp().addMinutes(lockPeriodInMinutes);
        }
    }

    @Override
    protected void blockStarted() {
		if(stakingToken == 0L || yieldToken == pledgeToken) {
			// stakingpool not initialized, do nothing
            // distributionToken = Token is not possible , never start
			return;
		}
        distributedAmount = ZERO;
        distributedQuantity = ZERO;
        distributionDone = false;
        contractActiveCheck = true;
        if(lockPeriodInMinutes > ZERO){
            lockPeriodTimeEnd = getBlockTimestamp().addMinutes(lockPeriodInMinutes);
            if(contractExpiryInMinutes > ZERO && lockPeriodTimeEnd.ge(stakingTimeout) ){
                lockPeriodTimeEnd = stakingTimeout;
            }
        }
        if(contractExpiryInMinutes > ZERO){
            if(stakingTimeout.le(getBlockTimestamp())){
                contractActiveCheck =false;
            }
        }
        while(true) {
            tx = getTxAfterTimestamp(lastProcessedTx);
            if(tx == null) {
				break;
			}
            arguments = tx.getMessage();
            lastProcessedTx = tx.getTimestamp();
            //Pledge Token check
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
                    //Send back pledgeToken as contract ended
                    sendAmount(pledgeToken,quantityCheck, tx.getSenderAddress());
                }
            }
            //Withdrawal staking Token check
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
                    if (lockPeriodInMinutes == ZERO){
                        sendAmount(pledgeToken, quantityCheck, tx.getSenderAddress());
                        totalstaked -= quantityCheck;
                        // burn stakingToken
                        sendAmount(stakingToken, quantityCheck, getAddress(ZERO));
                    }
                    else{
                        //send back stakingtokens
                        sendAmount(stakingToken, quantityCheck, tx.getSenderAddress());
                        //regsiter address 
                        setMapValue(FOUR,tx.getSenderAddress().getId(),lockPeriodTimeEnd.getValue());
                    }
                }
            }
            // Distribute airdropped tokens - calls the distribution method for airdropped tokens beside pledgeToken, stakingToken or yieldToken
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
            //Cleanup - after timeout and last payment the creator can get the balance 
            if (arguments.getValue1() == CLEANUP_BY_CREATOR && stakingTimeoutLastPayment &&  tx.getSenderAddress().equals(getCreator())){
                sendAmount(yieldToken,getCurrentBalance(yieldToken),getCreator());
                sendBalance(getCreator());
            }
        }
        blockheight = this.getBlockHeight();
        if(contractExpiryInMinutes == ZERO){
            if(blockheight - lastBlockDistributed >= paymentInterval ){
                checkRatio();
                distributeToStakingToken();
            }
        }
        else if(getBlockTimestamp().le(stakingTimeout)){
            if(blockheight - lastBlockDistributed >= paymentInterval ){
                checkRatio();
                distributeToStakingToken();
            }
        }
        else if(!stakingTimeoutLastPayment){
            checkRatio();
            distributeToStakingToken();
            // last payment is done - no payment anymore
            stakingTimeoutLastPayment = true;
        }

        // Storing current total staking
        setMapValue(ONE, blockheight, totalstaked);
        // store the distribution of SIGNA if any
        if (distributedAmount > ZERO && !distributionDone){
		    setMapValue(TWO, blockheight, distributedAmount);
        }
        // store distribution of distrubteToken if any
        if (distributedQuantity > ZERO && !distributionDone){
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
The creator is always the account which uploads the transaction for the creation of the staking contract to the chain.

### Variable setting (data stack)
While the creator is uploading the contract to the chain, the individual data stack for the staking contract should be defined within the transaction. Within the data stack, the values should be set in the following order:

**stakingTokenTicker**
<br>The ticker of the generated staking token by the contract with up to 10 numbers/letters.<br/>

**stakingTokenDecimals**
<br>The number of decimals for the staking token - should be the same as the token to stake.<br/>

**pledgeToken**
<br>Id of the token, which can be staked on the contract.<br/>

**digitsFactorPledgeToken**<
<br>The multiplier for the ratio calculation by digits of the **pledgetoken**<br/>
0 digits = 1 , 1 digit = 10 ... 8 digits = 100000000

**yieldToken**
<br>The Id of the token, which should be distributed by default -if a position exits on the contract.<br/>

**digitsFactorYieldToken**
<br>The multiplier for the ratio calculation by digits of the **yieldToken**<br/>
0 digits = 1 , 1 digit = 10 ... 8 digits = 100000000

**airdroppedTokenMinimumQuantity**
<br>Minimum quantity of a token to get distributed, which got airdropped to the contract.<br/>

Distribution parameter:

**paymentInterval**
<br>A minimum number of blocks the contract is waiting until the next distribution is executed.<br/>
If set to 0, no interval is checked.

**contractExpiryInMinutes**
<br>A value in minutes to calculate the expiry of the contract by creation.<br/>
If set to 0, the contract is infinity.

**qualifiedMinimumQuantity**
<br>The minimum quantity needed of the stakingToken to be eligible for a distribution.<br/>

**minAmountToDistributeSigna**
<br>The minimum amount of Signa required on the contract balance to trigger a payout.<br/>

**minQuantityToDistributeYieldToken**
<br>The minimum quantity of the **yieldToken** required on the contract balance to trigger a payout.<br/>

**maxAmountPerPayment**
<br>The maximum amount of Signa paid out at every **paymentInterval**.<br/>
If set to 0  the contract balance will be used to calculate the amount of Signa for a payout.

**maxQuantityPerPayment**
<br>The maximum quantity of **yieldToken** paid out at every **paymentInterval**.<br/>

**signaRatio**
<br>Besides a static **maxAmountPerPayment** a ratio can be defined as how many StakingTokens are needed to get one Signa. Depending on the number of existing StakingTokens the **maxAmountPerPayment** is calculated and set for each **paymentInterval**.<br/>
This function is inactive when set to 0.
Example: 100 stakedToken should get 1 Signa  means 100:1. signaRatio = 100

**tokenRatio**
<br>Besides a static **maxQuantityPerPayment** a ratio can be defined as how many StakingTokens are needed to get one **yieldToken**. Depending on the number of existing StakingTokens the **maxQuantityPerPayment** is calculated and set for each **paymentInterval**.<br/>
This function is inactive when set to 0.
Example: 1000 stakedToken getting 0.2 yieldToken means 200:1. tokenRatio = 200

**lockPeriodInMinutes**
<br>A value in minutes to calculate the lockout period for each **pledgeToken** by user address.<br/>
If this value is set to 0, no lockout period is calculated.

## Basic functions 
The contract will parse all transaction by the following basic logic:

#### Pledge Token check
The contract checks whether the transaction has transferred a **pledgeToken**.
If so, the same amount of **pledgeToken** is minted and sent to the sender of the transaction. If a **lockPeriodInMinutes** is set, a lock period for the address is stored in a map entry. Each time a new transfer is made from the same sender address, the lock period is overwritten with the latest calculation.

If the contract has already reached its expiry date - the **pledgeToken** will be just sent back to the sender.

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
If this is the case, the contract checks whether the lockup period has expired, sends back the same amount of **pledgeToken** and burns the received **stakingToken**.

If the contract has set a **lockPeriodInMinutes** and no map entry for the sender address was found, a new map entry will be set, and the **stakingToken** will be sent back to the sender.

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
                    if (lockPeriodInMinutes == ZERO){
                        sendAmount(pledgeToken, quantityCheck, tx.getSenderAddress());
                        totalstaked -= quantityCheck;
                        // burn stakingToken
                        sendAmount(stakingToken, quantityCheck, getAddress(ZERO));
                    }
                    else{
                        //send back stakingtokens
                        sendAmount(stakingToken, quantityCheck, tx.getSenderAddress());
                        //regsiter address 
                        setMapValue(FOUR,tx.getSenderAddress().getId(),lockPeriodTimeEnd.getValue());
                    }
                }
```
#### Distribute airdropped tokens
Any address can request to distribute airdropped tokens at any time.
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
#### Cleanup
Only when the contract has expired, and a final distribution has been made is the creator entitled to remove the current balances of Signa and **yieldToken** from the staking contract.
```java
            if (arguments.getValue1() == CLEANUP_BY_CREATOR && stakingTimeoutLastPayment &&  tx.getSenderAddress().equals(getCreator())){
                sendAmount(yieldToken,getCurrentBalance(yieldToken),getCreator());
                sendBalance(getCreator());
            }
  ```


### Creation Costs
The creation of a staking contract costs 2.60 Signa. 
To be paid with the deploy-contract transaction.

### Activation costs
The contract has an activation cost of 0.90 Signa. 
Every transaction needs to have this minimum amount to interact with it. Otherwise, the transaction will not be handled within the smart contract. 

### Ensure payment interval
To ensure that each payment interval is executed even if there is no activity on the staking contract, we recommend that the contract creator set up a subscription with a payment of 1.00 Signa at an interval equal to the **paymentInterval**.

## Compatibility
This new staking standard is compatible with the current smart contract framework. 
**No** hard fork needed.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

