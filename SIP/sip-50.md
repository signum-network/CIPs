---
sip: 50
title: Websockets API 
description: Definition of the Web Sockets API
author: ohager
status: Draft
type: standard
category: Interface, SRC
created: 2024-01-10
---


## Abstract
This document describes the lightweight real-time communication interface via Websockets for the Node Software.

## Motivation
The Node software currently offers an extensive HTTP interface to communicate remotely with the Node software. This interface is used by software applications such as wallets. Because HTTP is a pure request-response procedure, a client application must regularly ask for updates (polling). Websocket communication is used to establish a truly realtime connection between the client and the node software so that the server can notify connected clients directly about new blocks/transactions, for example. This allows communication to take place more efficiently according to the Hollywood principle, i.e. potentially less resource consumption (bandwidth, data volume).

## Specification

The new endpoint for the Websocket connection SHALL be available under `ws[s]://<host>:<port>/events`. Once a client connects to this endpoint, it should get different real-time events from the node software.


### Events

Following events MUST be emitted:

| Event             | Occurrance      | Purpose                                                 |
|-------------------|-----------|-------------------------------------------------------------|
| `CONNECTED`       | Immediate on connection    | Gives immediate feedback about successful connection and returns basic current state of the connected node |
| `HEARTBEAT`       | Periodically when Idle     | Informs about a standing, healthy connection. Can be used to detect interrupted connections              |
| `BLOCK_PUSHED`       | On each new block     | Informs that a new block was just added to the blockchain              |
| `PENDING_TRANSACTIONS_ADDED`       | On each incoming transaction     | Informs that a transactions was submitted to the network, waiting for confirmation |


Each event MAY have an additional payload.


#### Anatomy

The responses from the Node Software follow a unified structure. All responses MUST be serialized valid JSON (RFC8259)

| Field             | Type      | Description                                                 | Example                  |
|-------------------|-----------|-------------------------------------------------------------|--------------------------|
| `e`               | String    | Represents the event type or action. Event name SHOULD be uppercase. | `"CONNECTED"`            |
| `p`               | Object    | Payload containing relevant data for the event. This field is optional              |                          |

#### Event `CONNECTED`

The Node Software MUST emit this event upon every established Websocket connection.

The payload response SHOULD have at least the following fields:

| Field             | Type      | Description                                                 | Example                  |
|-------------------|-----------|-------------------------------------------------------------|--------------------------|
| `version`       | String    | Represents the version of the current node.         | `"v3.8.0-dev"`           |
| `networkName`   | String    | Represents the name of the network.                          | `"Signum-TESTNET"`       |
| `globalHeight`  | Integer   | Represents the global or overall height of the network.      | `632462`                 |
| `localHeight`   | Integer   | Represents the local or node-specific height in the network. | `315680`                 |
| `isConnected`   | Boolean   | Indicates whether the connection to the network is established, i.e. at least one peer fed a `globalHeight`| `true`                 |
| `isSyncing`     | Boolean   | Indicates whether the system is currently syncing with the network, i.e. `localHeight` is lower than `globalHeight`| `true`               |


_Example_

```json
{
  "e": "CONNECTED",
  "p": {
    "version": "v3.8.0-dev",
    "networkName": "Signum-TESTNET",
    "globalHeight": 632462,
    "localHeight": 315680,
    "isConnected": true,
    "isSyncing": true
  }
}

```


#### Event `HEARTBEAT`

The Node Software SHOULD emit this event periodically. It helps to identify if the connection to the Node Software is still established. 

This event has no payload.


_Example_

```json
{
  "e": "HEARTBEAT",
}

```

#### Event `BLOCK_PUSHED`

The Node Software SHALL emit this event upon new blocks, i.e. while syncing or while in sync. As the node can push several dozen to hundred blocks per second, the event emission MUST be debounced to a configurable duration, e.g. one second, where only the latest pushed block SHALL be submitted.

The payload response SHOULD have at least the following fields:

| Field                 | Type      | Description                                            | Example                  |
|-----------------------|-----------|--------------------------------------------------------|--------------------------|
| `timestamp`         | Integer   | Represents the epoch timestamp associated with the event.     | `223215201`              |
| `blockId`           | String    | Represents the unique identifier of the pushed block.   | `"7085823688029188009"` |
| `localHeight`       | Integer   | Represents the local or node-specific height.           | `335876`                 |
| `globalHeight`      | Integer   | Represents the global or overall height of the network. | `632424`                 |
| `progress`          | Float     | Represents the progress of the synchronisation progress, where [0 - not synced, 1 - fully synced]   | `0.53109306`             |
| `transactionCount`  | Integer   | Represents the number of transactions in this block   | `5`             |



_Example_

```json
{
  "e": "BLOCK_PUSHED",
  "p": {
    "timestamp": 223215201,
    "blockId": "7085823688029188009",
    "localHeight": 335876,
    "globalHeight": 632424,
    "progress": 0.53109306,
    "transactionCount": 5
  }
}
```

#### Event `PENDING_TRANSACTIONS_ADDED`

The Node Software SHALL emit this event when new transactions are added to the mem pool. 

The payload response SHOULD have at least the following fields:

| Field                    | Type      | Description                                                | Example                          |
|--------------------------|-----------|------------------------------------------------------------|----------------------------------|
| `transactionIds`       | Array     | Array of transaction IDs added in the pending transactions. | `["8987323561909124581"]`       |
| `hasMore`              | Boolean   | Optional indicator showing that there are more pending transactions.*      | `true`                          |

> * It is theoretically possible to have hundreds or thousands transactions in the array. To save bandwidth the array has to be capped, e.g. up to 100 tx. If there are more than the capped limit, the flag `hasMore` MUST be `true`, or MAY be omitted.

## Configuration

The following new configuration parameters for the `node.properties` are RECOMMENDED

```
# Enable or disable the Web Socket API
API.WebsocketEnable = true

# Port for Websocket/json API events (defaults to HTTP API Port + 1)
API.WebsocketPort = 8126

## The heartbeat interval in seconds that indicates a working connection (defaults to 30 seconds)
# API.WebsocketHeartbeatInterval = 30 
```

## Backwards Compatibility

This proposal is an extension of the existing features and has no impact on compatibility.

## Reference Implementation

https://github.com/signum-network/signum-node/pull/771


## Security Considerations

The Websocket connection is intended exclusively for reading public blockchain information. No transactions should be generated via Websockets. Nevertheless, it must be possible to establish a WSS connection for HTTPS connections, as otherwise all modern browsers do not allow _mixed content_ communication.


## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

