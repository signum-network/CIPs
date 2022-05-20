---
sip:  40
title: NFT
description: A standard NFT for Signum
author:   frank_the_tank
status: Review
type: standard
category: SRC
created: 2022-04-26
---
# Introduction
NFTs (non fungible token) are smart contracts representing an ownership of a digital asset. This SIP proposes a standard implementation of a smart contract to cover and track all needed activities around the ownership management. With  [SIP-30](sip-30.md)  the new standard can be simply used by linking the smart contract code from the first contract deployment of this SIP-40 standard. An audit check can be done by comparing the hash of the created NFT with the hash from this original contract deployment- if both are the same the original code is used.

SRC-40 Contract deployment : https://chain.signum.network/at/7074088231427605580

Transaction: 7074088231427605580

Code-Hash-ID:  15155055045342098571


# Motivation
The following standard allows the implementation of a standard API for NFTs within smart contracts. This standard provides basic functionality to track, trade  and transfer NFTs.

**What is a Non-Fungible Token?**

A Non-Fungible Token (NFT) is used to identify something or someone in a unique way. This type of Token is perfect to be used on platforms that offer collectible items, access keys, lottery tickets, numbered seats for concerts and sports matches, etc. This special type of Token has amazing possibilities, so it deserves a proper Standard, the SIP-40  came to solve that!

We considered use cases of NFTs being owned and transacted by individuals as well as consignment to third party brokers/wallets/auctioneers (“operators”). NFTs can represent ownership over digital or physical assets. We considered a diverse universe of assets:

-   Physical property — houses, unique artwork
-   Virtual collectables — unique pictures of kittens, collectable cards
-   “Negative value” assets — loans, burdens and other responsibilities


##  Java code for the SRC-40 contract
The Java code for the NFT standard looks as follows:

```java

    package bt.dapps;
    import bt.*;
    import bt.compiler.CompilerVersion;
    import bt.compiler.TargetCompilerVersion;
    import bt.ui.EmulatorWindow;
    
    /**
     * A Non-Fungible Token smart contract with sale and auction capabilities.
     * The contract has an owner that can transfer the ownership to another ddress.
     * Additionally, the owner can put the token up for sale or for auction with a
     * given timeout. The auction can have a max price set or it can also be a Dutch stile auction.
     * All operations are handled directly by the contract, so it is completely decentralized.
     * At every sale, the original creator can receive royalties and the platform can get a fee. The current royalties holder can also transfer the rights to another address (or contract).
     * All fees are configurable at the creation of the contract.
     * @author jjos
     * /
     
     @TargetCompilerVersion(CompilerVersion.v0_0_0)
     public class SignumArt2 extends Contract {
    
    // Main status variables
	Address owner;
	long status;
	long currentPrice;
	
	// Platform configuration
	Address platformAddress;
	long platformFee;
	long royaltiesFee;
	Address royaltiesOwner;
	
	// Tracker addresses
	Address trackSetNotForSale;
	Address trackSetForSale;
	Address trackAuctionOpened;	
	Address trackNewAuctionBid;
	Address trackNewOwner;
	Address trackOwnershipTransferred;
	Address trackDutchAuctionOpened;
	Address trackOfferReceived;
	Address trackOfferRemoved;
	Address trackLikeReceived;
	Address trackRoyaltiesTransfer;
	
	// Auction variables
	Timestamp auctionTimeout;
	Address highestBidder;
	long auctionMaxPrice;
	
	// Offer for an item even when it is not for sale/auction
	Address offerAddress;
	long offerPrice;
	
	// Dutch auction variables
	long duchStartHeight;
	long startPrice;
	long priceDropPerBlock;
	long reservePrice;

	// Additional statistics
	long totalTimesSold;
	long totalBidsReceived;
	long totalRoyaltiesFee;
	long totalPlatformFee;
	long totalLikes;
	
	// Intermediate variables
	long amountToRoyalties;
	long amountToPlatform;
	
	// Other constants
	private static final long ZERO = 0;
	private static final long STATUS_NOT_FOR_SALE = ZERO;
	private static final long STATUS_FOR_SALE = 1;
	private static final long STATUS_FOR_AUCTION = 2;
	private static final long THOUSAND = 1000;
	
	/** Use a contract fee of 0.3 SIGNA */
	public static final long CONTRACT_FEES = 30000000;

	/**
	 * Transfers the ownership of this token.
	 * Only the current owner can transfer the ownership.
	 * @param newOwner
	 */
	public void transfer(Address newOwner) {
		if (owner.equals(this.getCurrentTxSender())) {
			// only the current owner can transfer
			sendMessage(owner.getId(), newOwner.getId(), trackOwnershipTransferred);	
			owner = newOwner;
		}
	}
		/**
	 * Transfers the royalties ownership of this token.
	 * 
	 * Only the current royalties owner can transfer the ownership.
	 * 
	 * @param newRoyaltiesOwner
	 */
	public void transferRoyalties(Address newRoyaltiesOwner) {
		if (royaltiesOwner.equals(this.getCurrentTxSender())) {
			// only the current owner can transfer
			sendMessage(royaltiesOwner.getId(), newRoyaltiesOwner.getId(), trackRoyaltiesTransfer);	
			royaltiesOwner = newRoyaltiesOwner;
		}
	}

	/**
	 * Cancels an open for sale or auction and sets it as not for sale.
	 */
	public void setNotForSale(){
		if (highestBidder==null && owner.equals(this.getCurrentTxSender())) {
			// only if there is no bidder and it is the current owner
			status = STATUS_NOT_FOR_SALE;
			sendMessage(owner.getId(), trackSetNotForSale);
		}
	}

	/**
	 * Put this token for sale for the given price.
	 * 
	 * Buyer needs to transfer at least the asked amount.
	 * 
	 * @param priceNQT the price in NQT==1E-8 SIGNA (buyer needs to transfer at least
	 *                 this amount + gas fees)
	 */
	public void putForSale(long priceNQT) {
		if (highestBidder==null && owner.equals(this.getCurrentTxSender())) {
			// only if there is no bidder and it is the current owner
			status = STATUS_FOR_SALE;
			currentPrice = priceNQT;
			duchStartHeight = ZERO;
			sendMessage(owner.getId(), currentPrice, trackSetForSale);
		}
	}
	
	/**
	 * Put this token for a Duch auction style.
	 * 
	 * Buyer needs to transfer at least the current price, calculated as a function of block height.
	 * At every new block the price should be subtracted by the {@link #priceDropPerBlock} until
	 * the {@link #reservePrice} is reached.
	 * 
	 * @param startPrice the starting price
	 * @param priceDropPerBlock the amount to be subtracted from the price per block
	 * @param reservePrice the minimum accepted price
	 */
	public void putForDuchAuction(long startPrice, long reservePrice, long priceDropPerBlock) {
		if (highestBidder==null && owner.equals(this.getCurrentTxSender())) {
			// only if there is no bidder and it is the current owner
			status = STATUS_FOR_SALE;
			duchStartHeight = getBlockHeight();
			currentPrice = startPrice;
			this.startPrice = startPrice;
			this.priceDropPerBlock = priceDropPerBlock;
			this.reservePrice = reservePrice;
			
			sendMessage(owner.getId(), startPrice, reservePrice, priceDropPerBlock, trackDutchAuctionOpened);
		}
	}

	/**
	 * Put this token for auction with the minimum bid price and/or a max price
	 * 
	 * Bidders need to transfer more than current highest to become the new
	 * highest bidder. Previous highest bidder is refunded in case of a new
	 * highest bid arrives.
	 * 
	 * @param priceNQT the minimum bid price in NQT==1E-8 SIGNA (buyer needs to
	 *                 transfer at least this amount + contract fees)
	 * @param timeout  how many minutes the sale will be available
	 */
	public void putForAuction(long priceNQT, long maxPrice, int timeout) {
		if (highestBidder==null && owner.equals(this.getCurrentTxSender())) {
			// only if there is no bidder and it is the current owner
			status = STATUS_FOR_AUCTION;
			auctionTimeout = getBlockTimestamp().addMinutes(timeout);
			currentPrice = priceNQT;
			auctionMaxPrice = maxPrice;
			sendMessage(owner.getId(), currentPrice, auctionMaxPrice, auctionTimeout.getValue(), trackAuctionOpened);
		}
	}
	
	/**
	 * Make an offer for an item even if that is not for sale/auction.
	 * 
	 * This allows for potential buyers to make an offer even for items that are not for sale.
	 * The offer amount is locked in the contract until cancelled/accepted or a higher offer is received.
	 * 
	 * This offer has to be higher than a previous offer and can be cancelled later.
	 * The owner can accept the offer and the offer is cancelled/refunded if the item is
	 * actually sold or a running auction ends.
	 * 
	 */
	public void makeOffer() {
		if(getCurrentTxAmount() > offerPrice) {
			if(offerAddress != null) {
				// send back the latest offer
				sendAmount(offerPrice, offerAddress);
				sendMessage(offerAddress.getId(), offerPrice, trackOfferRemoved);
			}
			offerAddress = getCurrentTxSender();
			offerPrice = getCurrentTxAmount();
			sendMessage(offerAddress.getId(), offerPrice, trackOfferReceived);
			
			return;
		}
		// send back funds of an invalid order
		sendAmount(getCurrentTxAmount(), getCurrentTxSender());
	}
	
	/**
	 * Cancel a previously posted offer getting back the offer amount.
	 * 
	 * Only the offer maker can cancel an open offer.
	 */
	public void cancelOffer() {
		if(getCurrentTxSender().equals(offerAddress)) {
			cancelOfferIfPresent();
		}
	}
	
	private void cancelOfferIfPresent() {
		if(offerAddress == null)
			return;
		sendMessage(offerAddress.getId(), offerPrice, trackOfferRemoved);
		sendAmount(offerPrice, offerAddress);
		offerAddress = null;
		offerPrice = ZERO;
	}
	
	/**
	 * The owner accepts a posted offer.
	 */
	public void acceptOffer() {
		if(highestBidder==null && getCurrentTxSender().equals(owner) && offerPrice > ZERO) {
			currentPrice = offerPrice;
			pay();
			owner = offerAddress;
			offerAddress = null;
			offerPrice = ZERO;
			status = STATUS_NOT_FOR_SALE;
			sendMessage(owner.getId(), currentPrice, trackNewOwner);
		}
	}

	/**
	 * If this contract is for sale or for auction, this method handles the payment/bid.
	 * A buyer needs to transfer the asked price to the contract.
	 * If the token is for auction, a bidder needs to transfer more than the current highest
	 * bid to become the new highest bidder. Previous highest bidder is then refunded (minus
	 * the contract fee). After the auction timeout, any transaction received will trigger
	 * the ownership transfer.
	 * If the token was not for sale or the amount is not enough, the order is
	 * refunded (minus the contract fees).
	 */
	public void txReceived() {
		if (status == STATUS_FOR_SALE) {
			if (duchStartHeight > ZERO) {
				// Duch auction style, calculate the current price
				currentPrice = startPrice - (getBlockHeight() - duchStartHeight) * priceDropPerBlock;
				if(currentPrice < reservePrice) {
					currentPrice = reservePrice;
				}
			}
			if (getCurrentTxAmount() >= currentPrice) {
				// Conditions match, let's execute the sale
				pay(); // pay the current owner
				owner = getCurrentTxSender(); // new owner
				status = STATUS_NOT_FOR_SALE;
				sendMessage(owner.getId(), getCurrentTxAmount(), trackNewOwner);
				
				cancelOfferIfPresent();
				return;
			}
		}
		if (status == STATUS_FOR_AUCTION) {
			if (getBlockTimestamp().ge(auctionTimeout)) {
				// auction timed out, apply the transfer if any
				if (highestBidder != null) {
					pay(); // pay the current owner
					owner = highestBidder; // new owner
					highestBidder = null;
					status = STATUS_NOT_FOR_SALE;
					auctionMaxPrice = ZERO;
					sendMessage(owner.getId(), currentPrice, trackNewOwner);
					
					cancelOfferIfPresent();
				}
				// current transaction will be refunded below
			} else if (getCurrentTxAmount() > currentPrice) {
				// Conditions match, let's register the bid

				// refund previous bidder, if some
				if (highestBidder != null)
					sendAmount(currentPrice, highestBidder);

				highestBidder = getCurrentTxSender();
				currentPrice = getCurrentTxAmount();
				totalBidsReceived++;
				sendMessage( highestBidder.getId(), currentPrice, trackNewAuctionBid);
				
				if(auctionMaxPrice > ZERO && currentPrice >= auctionMaxPrice) {
					// max price reached, so we also end the auction
					pay(); // pay the current owner
					owner = highestBidder; // new owner
					highestBidder = null;
					status = STATUS_NOT_FOR_SALE;
					auctionMaxPrice = ZERO;
					sendMessage(owner.getId(), currentPrice, trackNewOwner);
					
					cancelOfferIfPresent();
				}
				return;
			}
		}
		// send back funds of an invalid order
		sendAmount(getCurrentTxAmount(), getCurrentTxSender());
	}
	
	public void likeIt() {
		totalLikes++;
		sendMessage(getCurrentTxSender().getId(), trackLikeReceived);
	}
	
	private void pay() {
		amountToPlatform = currentPrice * platformFee / THOUSAND;
		amountToRoyalties = currentPrice * royaltiesFee / THOUSAND;
		
		totalPlatformFee += amountToPlatform;
		totalRoyaltiesFee += amountToRoyalties;
		totalTimesSold++;
		
		sendMessage(owner.getId(), trackSetNotForSale);
		sendAmount(amountToRoyalties, royaltiesOwner);
		sendAmount(currentPrice - amountToPlatform - amountToRoyalties, owner);
	}
	
	protected void blockFinished(){
		// The platform fee is the remaining balance
		if(status == STATUS_NOT_FOR_SALE && offerAddress == null) {
			sendAmount(getCurrentBalance(), platformAddress);
		}
	}

```

### Creator of the NFT
The creator is always the account which uploads the transaction for the creation of the NFT to the chain.

### Variable setting (data stack)
While the creator is uploading the NFT to the chain, the individual data stack for the NFT should be defined within the transaction. Within the data stack the values should be set in the following order:

 * **owner**

	Should be equal creator by initial upload; can be set to any account id.

 * **status**

	Defines if the NFT is for sale. 
Valid values 
 	 *  0= Not for sale  
	 *  1= For sale at a fixed price 
	 *  2= For sale in an auction
	 
* **currentPrice**

	The current price of the NFT; should be zero  for status is 0 otherwise buy now price or floor price of the auction. Current price will change while an auction by valid bids.

* **platformAddress**

	The Signum account which will get the platformFee set under **platformFee**. This can also be used to filter for an NFT portal.

* **platformFee**

	PlatformFee in per mille. This value is send to the platformAddress by every sale.
2% is set as 20 in this field. (per mille)
The  platform fee is calculated from the sales price and reduce the payout to the current owner of the NFT on the active sale.

* **royaltiesFee**

	RoyaltiesFee in per mille. The royalties which are paid to the Royalties Owner by every sale of this NFT. 
10% is se as 100 in this field (per mille)
The royalties is calculated from the sales price and reduce the pay out to the current owner of the NFT on the active sale.

* **royaltiesOwner**

	The Signum account which will get the royaltiesFee by every sale of the NFT.

* **Tracker adresses**

	For each event the NFT will send a notification message to the defined Signum accounts:
	* trackSetNotForSale
	* trackSetForSale
	* trackAuctionOpened
	* trackNewAuctionBid
	* trackNewOwner
	* trackOwnershipTransferred
	* trackDutchAuctionOpened
	* trackOfferReceived
	* trackOfferRemoved
	* trackLikeReceived
	* trackRoyaltiesTransfer



## Basic functions 
The message structure is defined in each function calls.

### Transfer
With this function a current owner can transfer the NFT to a new owner.
A message is sent to the trackOwnershipTransferred account.

```java
     public void transfer(Address newOwner) {
		if (owner.equals(this.getCurrentTxSender())) {
			// only the current owner can transfer
			sendMessage(owner.getId(), newOwner.getId(), trackOwnershipTransferred);	
			owner = newOwner;
		    }
	    }
```

Method hash to call : `-8011735560658290665`

Argument 1 : AccountID (new owner)

### Transfer Royalties 
With this function a current owner of the royalties can transfer them to a new owner.
A message is sent to the trackRoyaltiesTransfer account.

```java
    public void transferRoyalties(Address newRoyaltiesOwner) {
		if (royaltiesOwner.equals(this.getCurrentTxSender())) {
			// only the current owner can transfer
			sendMessage(royaltiesOwner.getId(), newRoyaltiesOwner.getId(), trackRoyaltiesTransfer);	
			royaltiesOwner = newRoyaltiesOwner;
		  }
	  }
	  
```

Method hash to c all : `7174296962751784077`
Argument 1 : AccountID (new owner)

### Set not for sale
Setting the NFT not for sale anymore.
Aborting an auction is only possible if there is no current bid.
A message is sent to the trackSetNotForSale account.

```java
    public void setNotForSale(){
		if (highestBidder==null && owner.equals(this.getCurrentTxSender())) {
			// only if there is no bidder and it is the current owner
			status = STATUS_NOT_FOR_SALE;
			sendMessage(owner.getId(), trackSetNotForSale);
		   }
	    }
```

Method hash to call : `-1462395320800038545`
No arguments within the call.

### Put for sale
Setting the NFT for sale with a buy now price.
If there are no bids and the owner is the same as the sender of the transaction, the new sale is set with the specified price on the NFT. 
A message is sent to the trackSetForSale account.

```java
    public void putForSale(long priceNQT) {
	    if (highestBidder==null && owner.equals(this.getCurrentTxSender())) {
		    // only if there is no bidder and it is the current owner
		    status = STATUS_FOR_SALE;
		    currentPrice = priceNQT;
		    duchStartHeight = ZERO;
		    sendMessage(owner.getId(), currentPrice, trackSetForSale);    
		    }
	   }
```

Method hash to call : `483064598096680683`
Argument 1 : New Price (in Planck)

### Put for Dutch Auction
Setting the NFT for sale in a dutch auction style.
If no bids exists and the owner is equal the sender of the transaction, the new auction will be created for the NFT. By definition the Dutch Auction starts with a high price and drops over time to a floor price. If someone send a bid to the NFT which is above or equal the price over time thee auction will end and the bidder will get the NFT. A message is sent to the trackDutchAuctionOpened account.


```java
    public void putForDuchAuction(long startPrice,long reservePrice,long 
    priceDropPerBlock) 
      {
	    if (highestBidder==null && owner.equals(this.getCurrentTxSender())) {
	    // only if there is no bidder and it is the current owner
	    status = STATUS_FOR_SALE;
	    duchStartHeight = getBlockHeight();
	    currentPrice = startPrice;
	    this.startPrice = startPrice;
	    this.priceDropPerBlock = priceDropPerBlock;
	    this.reservePrice = reservePrice;
	    sendMessage(owner.getId(), startPrice, reservePrice, priceDropPerBlock, trackDutchAuctionOpened);
	    }
	  }
```

Method hash to c all : `8003219160642102093`

Argument 1 : Start Price (in Planck)

Argument 2 : Floor Price (in Planck)

Argument 3 : Drop in Price (in Planck)

The drop will be calculated for each block aka 4 minutes.

### Put for Auction
Setting the NFT for sale in an auction style.
If there are no bids and the owner is the same as the sender of the transaction, a new auction is created for the NFT. By definition, an auction always has a starting price, an auction end time and a buy-now price (optional).

While the auction is running the bidder with the highest bid will win the auction, when the auction will end. If a new bid arrives which is higher than the current one the bidder from this current bid will be refunded immediately. 

If a buy-now price exists and a new bid is received that is equal to or higher than this price, the sender will receive the NFT and the auction will end immediately.

A message is sent to the trackAuctionOpened account.

```java
    public void putForAuction(long priceNQT, long maxPrice, int timeout) {
	    if (highestBidder==null && owner.equals(this.getCurrentTxSender())) {
		    // only if there is no bidder and it is the current owner
		    status = STATUS_FOR_AUCTION;
		    auctionTimeout = getBlockTimestamp().addMinutes(timeout);
		    currentPrice = priceNQT;
		    auctionMaxPrice = maxPrice;
		    sendMessage(owner.getId(), currentPrice, auctionMaxPrice,auctionTimeout.getValue(), trackAuctionOpened);
		    }
	    }
```

Method hash to c all : `-1457630170795045271`

Argument 1 : Start/Floor Price (in Planck)

Argument 2 : Buy now  Price (in Planck) -> should be 0 if not set

Argument 3 : Auction run time  (in minutes)  


### Make an offer
This allows for potential buyers to make an offer even for NFTs that are not for sale. The offer amount is locked in the contract until cancelled/accepted or a higher offer is received. This offer has to be higher than a previous offer and can be cancelled later. The owner can accept the offer and the offer is cancelled/refunded if the item is actually sold or a running auction ends.

A message is sent to the trackOfferReceived account.

```java
    public void makeOffer() 
	    {
		    if(getCurrentTxAmount() > offerPrice) {
				    if(offerAddress != null) {
							  // send back the latest offer
						    sendAmount(offerPrice, offerAddress);
						    sendMessage(offerAddress.getId(), offerPrice, trackOfferRemoved);
					  }
					  offerAddress = getCurrentTxSender();
					  offerPrice = getCurrentTxAmount();
					  sendMessage(offerAddress.getId(), offerPrice, trackOfferReceived);
					  return;
				  }
			  // send back funds of an invalid order
			  sendAmount(getCurrentTxAmount(), getCurrentTxSender());
			}
```

Method hash to c all : `1966889121029848432`

No arguments within the call. The Signa amount send with the tranaction is equal the offer price. 

### Cancel an offer
This allows the current maker of the offer to cancel its offer and get refunded. Only when the sender of the transaction is equal to the current offer owner it will be executed.
A message is sent to the trackOfferRemoved account.

```java
    public void cancelOffer() {
		    if(getCurrentTxSender().equals(offerAddress)) {
		    cancelOfferIfPresent();
		    }
		 }

    private void cancelOfferIfPresent() {
		    if(offerAddress == null)
			    return;
			  sendMessage(offerAddress.getId(), offerPrice, trackOfferRemoved);
			  sendAmount(offerPrice, offerAddress);
			  offerAddress = null;
			  offerPrice = ZERO;
		 }
```

Method hash to c all : `-8537031161958386749`

No arguments within the call. 

### Accept an offer
This allows the current owner of the NFT to accept the offer. If an auction with an existing bid is still active the call will not succeed.  A message is sent to the trackNewOwner account.

```java
    public void acceptOffer() {
		    if(highestBidder==null && getCurrentTxSender().equals(owner) && offerPrice > ZERO) {
			    currentPrice = offerPrice;
			    pay();
			    owner = offerAddress;
			    offerAddress = null;
			    offerPrice = ZERO;
			    status = STATUS_NOT_FOR_SALE;
			    sendMessage(owner.getId(), currentPrice, trackNewOwner);
			    }
		}
```
    

Method hash to c all : `8330062149210158071`

No arguments within the call. 

### Set a Like
Any account can like the NFT and add +1 to the Like counter on the NFT by sending this  transaction to it.  A message is sent to the trackLikeReceived account.

```java
    public void likeIt() {
	    totalLikes++;
	    sendMessage(getCurrentTxSender().getId(), trackLikeReceived);
	   }
 ```

Method hash to c all : `-9009069050835256`

No arguments within the call. 

### Buying an NFT
With the normal send Signa transaction to the NFT (smart contract address) -  without a message  - any account can try to buy it or place a bid.  

### Activation costs
The contract has an activation cost of 0.3 Signa. Every transaction needs to have this minimum amount to interact with it, otherwise the transaction will not be handled within the smart contract. The activation cost(fee) will be subtracted from any bid/offer send to the NFT. 

Example: If you like to send a bid of 100 Signa you need to send an amount of 100.30 plus the network fee for the transaction.


## Compatibility
This new NFT standard is compatible with the current smart contract framework. 
**No** hard fork needed.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
