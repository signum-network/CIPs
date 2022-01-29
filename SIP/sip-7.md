---
sip:  7
title: Differential Unconfirmed Tx Propagation
description: Differential Unconfirmed Tx Propagation
author: PoCC/Brabantian
status: Final
type: Standard
category : Interface 
requires :  SIP-6
created: 2018-07-06
---

## Abstract
Currently, wallets poll their peers for UT's [SIP-6](sip-6.md) every 5 seconds. During each such poll they get all the UTs currently in mempool, which means that until a block is forged (240s on average), every UT in mempool can be potentially sent more than 20 times over and over again. Clearly this can cause a network load and could be mitigated with a more intelligent approach of sending UT's.

In a differential UT propagation, for every UT received each node attaches a timestamp to this UT with the "node time" it received this. When the node is asked by a peer to send unconfirmed transactions without such a timestamp, it sends all UT's in mempool and the timestamp of the newest UT of this package. The peer can remember this timestamp and next time it polls for UT's, it can send this timestamp with the request and gets only new UT's that arrived after this time.

## Motivation
Because of the expected growth in the number of nodes in the Signum network, together with a severe increase in the number of Unconfirmed Transactions, network conductivity has to be increased. One of the best ways to do this is by simply making sure that less data has to be transmitted.

An example of this can be found in the HTTP specification, where a "last modified" header can be given with a request, which will make sure the resource contents will only be transmitted in case it got modified since the given timestamp. By doing this, a server will have to process and send less data to the client, while the client won't have to receive and process the same data twice.

A comparable solution is suggested in this SIP, where a peer should be able to ask another peer for only the UTs the fellow peer received after a specified timestamp. For reverse compatibility, and because of the need of an initial timestamp, there should still be the possibility to get all UTs without specifying any timestamp.

The maximum amount of UTs held in the UTS [SIP-6](sip-6.md) can be configured in the wallet configuration. In case a peer would ask it for all UTs it has, this might result in a too big result, potentially triggering a blacklisting of the bigger peer. In order to ask for more "bite-size chunks" an additional parameter should be expected that defines the max number of UTs returned. The combination of all the returned UTs of several of these calls to the same peer should of course be the same set in case the call was done once for the full set of UTs, so no UT will be forgotten.

## Specification
### Wallet Internals

The UTs in the UTS are required to have an additional field - or be wrapped together with a field - that will specify the moment they got added to the UTS. The standard implementation of this is the UnconfirmedTransactionTiming, which wraps together a timestamp and transaction. These will be saved in the internal storage as described in [SIP-6](sip-6.md).
```java
class UnconfirmedTransactionTiming {

  private final Transaction transaction;
  private final long timestamp;

  /* .. */
}
```

The UTS has an operation that will return a list of all the UTs it currently holds. This result will need to be extended with a timestamp that will contain either the current timestamp in case no UTs are returned, or otherwise the highest timestamp of the UTs found in the result.

A second operation needs to be added as well, which will give a similar result, but takes two parameters: timestamp, and limit. The timestamp parameter will define the minimum timestamp a UT should contain to be returned. The limit parameter will define the maximum amount of UTs returned by the method. Since the timestamp returned with this method will be either the current one or the highest one of the returned set of UTs, this timestamp can be reused in following requests to get a following set of UTs.

An example implementation of this operation is the following:

```java
public TimedUnconfirmedTransactionOverview getAllSince(long timestampInMillis, int limit) {
  synchronized (internalStore) {
    final ArrayList<UnconfirmedTransactionTiming> flatTransactionList = new ArrayList<>();

    for (List<UnconfirmedTransactionTiming> amountSlot : internalStore.values()) {
      flatTransactionList.addAll(
        amountSlot.stream()
        .filter(t -> t.getTimestamp() > timestampInMillis)
        .collect(Collectors.toList())
      );
    }

    final List<UnconfirmedTransactionTiming> result = flatTransactionList.stream()
        .sorted(Comparator.comparingLong(UnconfirmedTransactionTiming::getTimestamp))
        .limit(limit)
        .collect(Collectors.toList());

    if(! result.isEmpty()) {
      return new TimedUnconfirmedTransactionOverview(result.get(result.size() - 1).getTimestamp(),
        result.stream()
        .map(UnconfirmedTransactionTiming::getTransaction)
        .collect(Collectors.toList())
      );
    } else {
      return new TimedUnconfirmedTransactionOverview(timeService.getEpochTimeMillis(), new ArrayList<>());
    }
  }
}
```

### HTTP Peers layer

The additional parameters should be passed along from the HTTP Peers layer, added as additional URL parameters. These parameters will be known there as "lastUnconfirmedTransactionTimestamp" which defines the timestamp, and "limitUnconfirmedTransactionsRetrieved" which will be used for the limit. Along with the regular result of UTs from this HTTP call, an additional result  "lastUnconfirmedTransactionTimestamp" will be returned in the message body. This will contain the timestamp returned from the UTS.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
