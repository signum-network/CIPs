---
sip:  49
title: Liquidty Pool Contracts
description: A standard for Liquidity Pool Contracts
author:   frank_the_tank
status: Review
type: standard
category: SRC
created: 2023-10-15
updated: 2023-10-15
---
# Introduction
Liquidity pool contracts provide a platform for users to contribute their assets, such as token X and token Y, into a shared pool. Users can participate by adding liquidity to the pool, allowing them to earn rewards in the form of a special liquidity token, denoted as "XY." This liquidity pool contract facilitates the swapping of tokens X and Y while ensuring fairness and security in the transactions. Users can engage in swaps from token X to token Y and vice versa, and the contract manages fees and slippage to maintain the integrity of the pool.

With the introduction of the [SIP-30](sip-30.md), this new liquidity pool standard can be easily integrated into the Signum ecosystem. The deployment of the initial contract following SIP-49 can be audited by comparing its hash with the hash of the created Liquidity Pool Contract. If both hashes match, it confirms the use of the original code, ensuring the contract's authenticity and security.


SRC-49 Contract deployment:https://chain.signum.network/tx/2643236083511734871

Transaction: 2643236083511734871

Reference Hash: 573236a736a9ae24b1e66113a23f86d1a8f1f54e267ad0cd570c47eaaccf3b84

Code-Hash-ID: 2588253152591293552

## Abstract

The Java code presented in this contract defines an Automated Market Maker (AMM) or liquidity pool smart contract designed for the Signum blockchain. The contract facilitates the management of liquidity pools for two distinct tokens, referred to as token X and token Y or Signa and another token. Users can add liquidity to the pool and, in return, receive a special token, token XY aka Liquidity Pool token (LP token), which can be used like any other smart token. Liquidity can also be removed by sending back the token XY.

The key features of this contract include:

1. **Liquidity Management:** <br>
Users can add and remove liquidity to the pool, and the contract automatically mints and burns token XY in response to these actions. The liquidity addition mechanism ensures that the price of the tokens remains fair and that slippage is minimal.

2. **Protection Against Sandwich Attacks:** <br>
The contract is designed to prevent the "sandwich attack," ensuring that all swaps within a given block pay the same price and that all liquidity addition and removal operations precede any trades. This design enhances the security and fairness of the liquidity pool. This also means that swaps outside the price range within the block will be rejected, which makes it imoppsible for scripts/bots to generate any arbitrage profit.

3. **Dynamic Swap Fees:** <br>
The contract allows for dynamic adjustment of swap fees, providing flexibility to adapt to changing market conditions and user preferences.

4. **Platform Fees:** <br>
The contract supports the collection of platform fees, enabling the platform to generate revenue from the liquidity pool's activities.

5. **Security Measures:** <br>
The contract includes various security measures to safeguard user assets and ensure that the platform operates smoothly.

This AMM contract offers a robust and secure solution for managing liquidity pools on the Signum blockchain, catering to a wide range of DeFi use cases and trading scenarios.

# Motivation

The introduction of Liquidity Pool Smart Contracts on the Signum chain, as outlined in this Signum Improvement Proposal (SIP), is driven by compelling motivations to enhance and expand the Signum ecosystem. These motivations revolve around addressing specific issues, offering novel solutions, and delivering substantial benefits to the Signum community.

1. **Enhancing Liquidity** <br>
The primary motivation behind this SIP is to provide Signum users with a dedicated platform for creating and managing liquidity pools. Liquidity is the lifeblood of any blockchain ecosystem, and by enabling users to contribute assets to these pools seamlessly, we ensure that Signum remains a vibrant and dynamic platform for decentralized finance (DeFi) applications. Liquidity pool smart contracts empower users to participate in yield farming, lending, and other financial activities, increasing the attractiveness of Signum as a DeFi hub.

2. **Reducing Complexity** <br>
Liquidity pool smart contracts eliminate the need for intermediaries, simplifying the process of providing liquidity and earning rewards. This SIP's motivation is to streamline DeFi participation by enabling direct peer-to-peer interactions, removing unnecessary barriers, and reducing the costs associated with traditional financial services.

3. **Empowering Developers** <br>
By standardizing Liquidity Pool Smart Contracts on Signum, this proposal motivates developers to build and innovate within the ecosystem. These contracts serve as a foundational building block for a wide range of DeFi applications, encouraging developers to create novel financial products, decentralized exchanges, and lending platforms, which can help foster a robust developer community and enhance Signum's utility.

4. **Global Accessibility** <br>
Signum's motivation for introducing Liquidity Pool Smart Contracts aligns with the broader goal of making DeFi accessible to a global audience. By enabling anyone with an internet connection to participate in these pools, Signum can promote financial inclusion and create new opportunities for those traditionally underserved by the traditional banking system.

5. **Incentivizing Growth** <br>
The implementation of Liquidity Pool Smart Contracts motivates growth within the Signum ecosystem. Users get incentivized to participate by earning rewards through yield farming, staking, or providing liquidity, ultimately fueling the development of the platform's user base and ecosystem. In turn, it increases the adoption and value of Signum's native tokens.

6. **Solving the Sandwich Problem:** <br>
One of the key innovations that this SIP brings to the Signum ecosystem is the solution to the "sandwich problem." By ensuring that all users get the same price within a single block, this innovation eliminates the arbitrage opportunities and market inefficiencies that traders can exploit, thus creating a fair and transparent trading environment for all participants.

In conclusion, the motivation behind introducing Liquidity Pool Smart Contracts as a standard on the Signum platform is driven by a combination of factors that collectively aim to strengthen the ecosystem. By addressing liquidity, reducing friction, empowering developers, enhancing security, promoting global accessibility, incentivizing growth, and solving the "sandwich problem," this SIP strives to make Signum an attractive hub for DeFi activities, fostering innovation and enriching the user experience.


##  Java code for the SRC-45 contract
The Java code for the staking standard looks as follows:

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
 * An Automated Market Maker (AMM) or liquidity pool smart contract.
 * 
 * There is a pair of token X and token Y that are kept inside the contract.
 * 
 * When adding liquidity an investor gets back the token XY. This token can
 * be used just as a regular token. The investor can later remove liquidity
 * by sending back the XY tokens.
 * 
 * About the liquidity token, XY, it is issued by the contract when it first
 * runs. Be sure to send at least the SIGNA amount for the token issuance fee
 * in a first transaction.
 * 
 * New tokens are minted by code when liquidity is added and burnt when liquidity is
 * removed.
 * 
 * The code is "shielded", not allowing the "sandwich attack", with all swaps
 * in a given block paying the same price and all liquidity addition/removal going
 * before any trades.
 * 
 * @author jjos, frank_the_tank
 *
 */
public class ShieldSwap extends Contract {
	//Variables to setup LP	
	long name;
	long tokenX;
	long tokenY;
	// portal fee address
	Address platformContract;
	//address to change swap fee for LPs
	Address swapFeeAddress;
	boolean isSwapFeeDynamic;
	// Fee 1% = 100
	long swapFee;
	long platformFeeSet;

	// Internal variabels used by the contract
	long tokenXY;
	long reserveX;
	long reserveY;
	long totalSupply;
	
	long reserveXBlock, reserveYBlock;
	long priceTimesReserveMaxX, priceTimesReserveMaxY, priceTimesReserve;
	Timestamp lastProcessedLiquidity;
	Timestamp lastProcessedSwapCheck;
	Timestamp lastProcessedSwap;
	long platformFee;
	long platformFeeBlockX;
	long platformFeeBlockY;
	long lpFeeBlockX;
	long lpFeeBlockY;
	long swapVolumeX;
	long swapVolumeY;
	
	Transaction tx;
	Register arguments;
	boolean txApproved;
	long minOut;
	
	// temporary variables
	long dx, dy;
	long liquidity;
	long liquidity2;
	long fee, x1, y1;
	long slippage;
	long checkFirstLP;
	long checkFirstLPFlag;
	Address SenderAccount; 
	long TransactionId;
	// We want the sqrt, so power is 0.5 = 5000_0000 / 10000_0000;
	private static final long SQRT_POW = 5000_0000;

	private static final long KEY_PROCESS_SWAP = 0;
	private static final long KEY_RESERVE_X = 1;
	private static final long KEY_RESERVE_Y = 2;
	private static final long KEY_LP_FEE_X  = 3;
	private static final long KEY_LP_FEE_Y  = 4;
	private static final long KEY_PF_FEE_X  = 5;
	private static final long KEY_PF_FEE_Y  = 6;
	private static final long KEY_SWAP_X_VOLUME  = 7;
	private static final long KEY_SWAP_Y_VOLUME  = 8;
	private static final long KEY_REMOVE_LP_X  = 9;
	private static final long KEY_REMOVE_LP_Y  = 10;
	private static final long KEY_SWAP_XY_Y  = 11;
	private static final long KEY_SWAP_YX_X  = 12;


	public static final long ADD_LIQUIDITY_METHOD = 1;
	public static final long REMOVE_LIQUIDITY_METHOD = 2;
	public static final long SWAP_XY_METHOD = 3;
	public static final long SWAP_YX_METHOD = 4;
	public static final long CHANGE_SWAP_FEE = 10;
	
	public static final long ZERO = 0;
	public static final long ONE = 1;
	public static final long TWO = 1;
	public static final long TENTHOUSAND = 10000;	
	public static final long THOUSAND = 1000;
	public static final long minSlippage= 1001;	
	// 1001 means minSlippage needs to be 0.1%
	private static final long LP_CHECK_1 = 10000000;
	private static final long LP_CHECK_2 = 1000000;
	private static final long LP_CHECK_3 = 100000;

	public ShieldSwap() {
		// constructor, runs when the first TX arrives
		tokenXY = issueAsset(name, 0L, ZERO);
	}
	
	/**
	 * We process all the swap transactions that will be approved so all swaps will pay the same price.
	 * This avoids the "sandwich attack" present in most liquidity pools available today.
	 */
	@Override
	protected void blockStarted() {
		if(tokenXY == 0L && tokenX == tokenY){
			// pool not initialized, and token x and y are the same (for any reason) do nothing
			return;
		}
		// First we iterate to add/remove liquidity
		while(true) {
			tx = getTxAfterTimestamp(lastProcessedLiquidity);
			if(tx == null) {
				break;
			}
			lastProcessedLiquidity = tx.getTimestamp();
			arguments = tx.getMessage();
			// we check also for a swapfee change
			if ((arguments.getValue1() == CHANGE_SWAP_FEE) && tx.getSenderAddress() == swapFeeAddress && isSwapFeeDynamic){
				if(arguments.getValue2() >= ZERO){
					swapFee = arguments.getValue2();
				}
			}
			
			if(arguments.getValue1() == ADD_LIQUIDITY_METHOD) {
				dx = tx.getAmount(tokenX);
				dy = tx.getAmount(tokenY);
				SenderAccount = tx.getSenderAddress();
				if(totalSupply == ZERO) {
					checkFirstLPFlag = ZERO;
					liquidity = calcPow(dx, SQRT_POW)*calcPow(dy, SQRT_POW);
					checkFirstLP = liquidity/LP_CHECK_1;
					if (checkFirstLP > ZERO ){
						liquidity = checkFirstLP;
						checkFirstLPFlag = ONE;
					}
					checkFirstLP = liquidity/LP_CHECK_2;
					if (checkFirstLP > ZERO && checkFirstLPFlag == ZERO ){
						liquidity = checkFirstLP;
						checkFirstLPFlag = ONE;
					}
					checkFirstLP = liquidity/LP_CHECK_3;
					if (checkFirstLP > ZERO && checkFirstLPFlag == ZERO ){
						liquidity = checkFirstLP;
					}
				}
				else {
					liquidity = calcMultDiv(dx, totalSupply, reserveX);
					liquidity2 = calcMultDiv(dy, totalSupply, reserveY);
					if(liquidity2 < liquidity)
						liquidity = liquidity2;
				}
				
				mintAsset(tokenXY, liquidity);
				sendAmount(tokenXY, liquidity, SenderAccount);
				totalSupply = totalSupply + liquidity;
				reserveX += dx;
				reserveY += dy;
			}
			else if(arguments.getValue1() == REMOVE_LIQUIDITY_METHOD) {
		        liquidity = tx.getAmount(tokenXY);
				SenderAccount = tx.getSenderAddress();
				TransactionId = tx.getId();
		        dx = calcMultDiv(liquidity, reserveX, totalSupply);
		        dy = calcMultDiv(liquidity, reserveY, totalSupply);
		        totalSupply = totalSupply - liquidity;
		        reserveX -= dx;
		        reserveY -= dy;
		        
		        sendAmount(tokenX, dx,SenderAccount);
		        sendAmount(tokenY, dy,SenderAccount);
		        // burn the XY token
		        sendAmount(tokenXY, liquidity, getAddress(ZERO));
				//Connect payout with original tx
				setMapValue(KEY_REMOVE_LP_X, TransactionId, dx);
				setMapValue(KEY_REMOVE_LP_Y,TransactionId, dy);
		    }
		}

		// Now we iterate to check which swaps should be accepted and what should be
		// the reserve changes within the block
		reserveXBlock = reserveX;
		reserveYBlock = reserveY;
		priceTimesReserveMaxX = ZERO;
		priceTimesReserveMaxY = ZERO;
		platformFeeBlockX = ZERO;
		platformFeeBlockY = ZERO;
		lpFeeBlockX = ZERO;
		lpFeeBlockY = ZERO;
		swapVolumeX = ZERO;
		swapVolumeY = ZERO;

		while(true) {
			tx = getTxAfterTimestamp(lastProcessedSwapCheck);
			if(tx == null) {
				break;
			}
			lastProcessedSwapCheck = tx.getTimestamp();
			
			if(totalSupply == ZERO) {
				// no liquidity to operate
				continue;
			}
			// Set dx and dy to ZERO otherwise double count at line 294,295
			dx = ZERO;
			dy = ZERO;
			txApproved = false;
			arguments = tx.getMessage();
			minOut = arguments.getValue2();
			if(minOut > ZERO) {
				if(arguments.getValue1() == SWAP_XY_METHOD) {
					dx = tx.getAmount(tokenX);
					fee = calcMultDiv(dx, swapFee,TENTHOUSAND);
					platformFee =calcMultDiv(dx, platformFeeSet,TENTHOUSAND);
					x1 = reserveXBlock + dx;
					y1 = calcMultDiv(reserveXBlock, reserveYBlock, x1 - fee - platformFee);

					dy = y1 - reserveYBlock;
					priceTimesReserve = calcMultDiv(dx, reserveY, minOut);
					
					if(-dy >= minOut && priceTimesReserve > ZERO) {
						if (priceTimesReserveMaxX == ZERO) {
							// first accepted swap in this direction, check for a minimum slippage
							slippage = calcMultDiv(priceTimesReserve, THOUSAND, reserveX);
							if(slippage < minSlippage) {
								// below minimum slippage
								continue;
							}
							priceTimesReserveMaxX = priceTimesReserve;
						}
						if (priceTimesReserve <= priceTimesReserveMaxX){
							txApproved = true;
							platformFeeBlockX += platformFee;
							lpFeeBlockX += fee;
							swapVolumeX += dx;
						}
					}
				}
				else if(arguments.getValue1() == SWAP_YX_METHOD) {
					dy = tx.getAmount(tokenY);
					
					fee = calcMultDiv(dy, swapFee,TENTHOUSAND);;
					platformFee = calcMultDiv(dy, platformFeeSet,TENTHOUSAND);
					y1 = reserveYBlock + dy;
					x1 = calcMultDiv(reserveXBlock, reserveYBlock, y1 - fee - platformFee);
					
					dx = x1 - reserveXBlock;
					priceTimesReserve = calcMultDiv(dy, reserveX, minOut);

					if(-dx >= minOut && priceTimesReserve > ZERO) {
						if (priceTimesReserveMaxY == ZERO) {
							// first accepted swap in this direction, check for a minimum slippage
							slippage = calcMultDiv(priceTimesReserve, THOUSAND, reserveY);
							if(slippage < minSlippage) {
								// below minimum slippage
								continue;
							}
							priceTimesReserveMaxY = priceTimesReserve;
						}
						if (priceTimesReserve <= priceTimesReserveMaxY){
							txApproved = true;
							platformFeeBlockY += platformFee;
							lpFeeBlockY += fee;
							swapVolumeY += dy;
						}
					}
				}
				
				if(txApproved) {
					// Update the amount exchanged and store the tx as processed
					reserveXBlock += dx;
					reserveYBlock += dy;
					setMapValue(KEY_PROCESS_SWAP, tx.getId(), minOut);
				}
			}			
		}
		
		// finally, we execute the accepted swaps with the liquid changes, all paying the same price
		while(true) {
			tx = getTxAfterTimestamp(lastProcessedSwap);
			if(tx == null) {
				break;
			}
			lastProcessedSwap = tx.getTimestamp();
			SenderAccount = tx.getSenderAddress();
			TransactionId =tx.getId();
			arguments = tx.getMessage();
			minOut = arguments.getValue2();
			if(arguments.getValue1() == SWAP_XY_METHOD || arguments.getValue1() == SWAP_YX_METHOD) {
				if(getMapValue(KEY_PROCESS_SWAP, tx.getId()) == ZERO) {
					// this swap was not approved, refund
					sendAmount(tokenX, tx.getAmount(tokenX), SenderAccount);
					sendAmount(tokenY, tx.getAmount(tokenY), SenderAccount);
				}
				else {
					if(arguments.getValue1() == SWAP_XY_METHOD) {
						dx = tx.getAmount(tokenX);
						fee = calcMultDiv(dx, swapFee,TENTHOUSAND)+ calcMultDiv(dx, platformFeeSet,TENTHOUSAND);
						dx -= fee;
						dy = calcMultDiv(-dx, reserveY, reserveXBlock);
							
						sendAmount(tokenY, -dy, SenderAccount);
						setMapValue(KEY_SWAP_XY_Y, TransactionId, -dy);
					}
					else {
						// swap YX
						dy = tx.getAmount(tokenY);
						
						fee = calcMultDiv(dy, swapFee,TENTHOUSAND) + calcMultDiv(dy, platformFeeSet,TENTHOUSAND);
						dy -= fee;
						dx = calcMultDiv(-dy, reserveX, reserveYBlock);
						
						sendAmount(tokenX, -dx, SenderAccount);
						setMapValue(KEY_SWAP_YX_X, TransactionId, -dx);
					}
				}
			}
		}
		if(platformFeeBlockX > ZERO) {
			sendAmount(tokenX, platformFeeBlockX, platformContract);
		}
		if(platformFeeBlockY > ZERO) {
			sendAmount(tokenY, platformFeeBlockY, platformContract);
		}
		// store the platform fee on this block
		setMapValue(KEY_PF_FEE_X ,this.getBlockHeight(), platformFeeBlockX);
		setMapValue(KEY_PF_FEE_Y, this.getBlockHeight(), platformFeeBlockY);
		// store the  swap volume X and Y
		setMapValue(KEY_SWAP_X_VOLUME ,this.getBlockHeight(), swapVolumeX);		
		setMapValue(KEY_SWAP_Y_VOLUME ,this.getBlockHeight(), swapVolumeY);	
		// store the lp fee for x and Y
		setMapValue(KEY_LP_FEE_X ,this.getBlockHeight(), lpFeeBlockX);		
		setMapValue(KEY_LP_FEE_Y ,this.getBlockHeight(), lpFeeBlockY);		
		// update the reserves when the block finishes to reconcile any dust/revenue
		reserveX = this.getCurrentBalance(tokenX);
		reserveY = this.getCurrentBalance(tokenY);
		// store the price on this block
		setMapValue(KEY_RESERVE_X, this.getBlockHeight(), reserveX);
		setMapValue(KEY_RESERVE_Y, this.getBlockHeight(), reserveY);
		if(totalSupply == ZERO && tokenXY != 0L ){
			if(tokenX != 0L && tokenY != 0L ) {
				dx =  getCurrentBalance() - getActivationFee();
				if (dx > ZERO) {
					sendAmount(dx, platformContract);
				}
			}
		}
	}
	
	/**
	 * This method removes/cleans-up an unwanted token.
	 * 
	 * @param tokenId the token ID
	 */
	public void cleanToken(long tokenId) {
		if(tokenId == 0L || tokenId == tokenX || tokenId == tokenY) {
			// invalid, so we do nothing
			return;
		}
		sendAmount(tokenId, getCurrentBalance(tokenId), platformContract);
	}
	
	/**
	 * Allows to update the platform contract account.
	 * 
	 * @param newPlatformContractId
	 */
	public void upgradePlatformContract(long newPlatformContractId) {
		if(getCurrentTxSender() == platformContract) {
			// only the current platform can upgrade itself to a new account
			platformContract = getAddress(newPlatformContractId);
		}
	}

	@Override
	public void txReceived() {
		// do nothing
	}
}
```

### Creator of the Liquidity Pool Contract
The creator is always the account which uploads the transaction to create the liquidity pool contract to the chain.

### Variable setting (data stack)
The Uploader should set the data stack within the creation transaction of the smart contract. In the data stack, the data needs to be set  in the following order:

**name**<br>
The ticker of the generated liquidity token by the contract with up to 10 numbers/letters.<br/>

**TokenX**<br>
The ID of the token is the first token of the token pair. 
It should be 0 for Signa<br/>


**TokenY**<br>
The ID of the token is the second token of the token pair.
It should be 0 for Signa<br/>


**platformContract**<br>
An account ID which receives the platform fee set in the variable **platformFeeSet** .<br/>

**swapFeeAddress**<br>
The account which is able to change the SwapFee if **isSwapFeeDynamic** is set to 1 (true)<br/>

**isSwapFeeDynamic**<br>
Setting to 1 or 0 (true/false). If true the  defined account under **swapFeeAddress** is able to change the **swapFee** <br/>

**swapFee**<br>
The swapFee for the liquidity provider is taken from the swap initiator. The value is set at 100- which means 100 equals 1%<br/>

**platformFeeSet**<br>
The platformfee is sent to the **platformContract** . The value is set at 100- which means 10 equals 0.1%<br/>

## Contract logical loops
The contract will parse all transactions by the following basic logic:

1.  **First, the Liquidity Processing Loop:**    
    -   The loop iterates through transactions to add or remove liquidity.
    -   If it's an "Add Liquidity" transaction, the code calculates the liquidity and issues "XY" tokens to the user.
    -   If it's a "Remove Liquidity" transaction, the code calculates how many tokens X and Y the user will receive. It effectively burns the "XY" tokens used.
    -   The loop ensures that these liquidity-related transactions are processed before swaps.
2.  **Swap Processing Loop:**
      -   After handling liquidity transactions, the code enters the swap processing loop.
    -   For each transaction, the code first checks for liquidity in the pool. If not, it skips the transaction.
    -   For each swap transaction, it checks whether it's a valid swap from token X to token Y or from token Y to token X.
    -   It calculates fees associated with the swap, including swap and platform fees.
    -   The code checks if the swap meets specific criteria, such as minimum slippage, before approval.
    -   If the swap is approved, the code updates token X and Y reserve levels, records the transaction as processed, and applies fees accordingly.
3.  **Platform Fee and Liquidity Provider Fee Processing:**
       -   After processing the swaps, the code calculates platform and liquidity provider fees for both directions (X to Y and Y to X).
    -   It records the current block's platform and liquidity provider fees, ensuring that the fees are collected and distributed fairly.
4.  **Reserve and Price Updates:**
       - After processing all swap transactions in the block, the code updates the reserve amounts for token X and token Y to reconcile any dust or revenue.
    -   It stores the reserve levels and prices at the end of the block, maintaining an accurate record of the pool state.

This sequence ensures that liquidity-related transactions are processed first, followed by swap transactions with careful consideration of fees, slippage, and pricing. The code maintains a fair and secure environment for liquidity providers and users participating in swaps within the pool.

### Creation Costs
The creation of a staking contract costs 3.50 Signa. To be paid with the deploy-contract transaction. After creating the smart contract, the creator needs to send a second transaction with 152 Signa - to initiate the LP token on the contract.

### Activation costs
The contract has an activation cost of 1.80 Signa. 
Every transaction needs to have this minimum amount to interact with it. Otherwise,  the smart contract will not handle it.


## Compatibility
This new liquidity pool standard is compatible with the current smart contract framework. 
**No** hard fork needed.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
