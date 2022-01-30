---
sip:  14
title: Signum Actions through Deeplink QR-codes
description: Signum Actions through Deeplink QR-codes
author: PoCC/Brabantian
status: Final
type: Standard
categroy: Interface
requires: SIP-13
created: 2018-08-19
---
## Abstract

[QR codes](https://en.wikipedia.org/wiki/QR_code) with [deeplinks](https://en.wikipedia.org/wiki/Deep_linking) enable users to scan a QR code, and be automatically led into a specific screen of an application of a mobile device. This deeplink is similar to a hyperlink we know from webpages. It contains a protocol refering to the application specified, and it can encode extra information on which screen it needs to lead to. This enables a very interactive experience using these QR codes, to improve the user experience of Signum.

## Motivation

One of the main goals of SIgnum is to be a monetary unit that's easily exchanged between different users. It is to be expected that most person-to-person Signum transactions will happen in stores. A client purchases an amount of items, and wishes to pay for it with Signa.

To facilitate this better, a Point of Sales (PoS) software should exist for the store to use, itemizing the different purchases, and adding them together to the sum that has to be paid by the client. After this, a QR code can be generated, which the client has to scan with their QR scanner, to lead them to the "payment" screen in their Signum mobile application. This screen will have a number of predefined values for the amount of Signa that has to be paid, the suggested fee, and an extra message (to link the transaction back to the internal DB of the PoS software), all defined by the PoS software.

As not all payments will happen in stores, an another use case is when one user wants to pay another one. Generating a QR code can improve the user experience of conducting such a transaction. Instead of requiring the sender to type in the recipient's Signum address, the recipient QR code can be scanned, and the remaining fields filled manually.

The capability improvement is not limited to usage in situations where the sender and the recipient are in the same location. Recipient QR codes can be published in various locations: e.g. an artist publishes their QR code next to a piece of art they created - enabling art lovers to give a donation by scanning the  QR code as decsribed above.

## Specification

### Wallet internals
To facilitate the QR code generation, a new dependency is added to the [Zebra Crossing](https://github.com/zxing) library,
falling under the [Apache License v2.0](http://www.apache.org/licenses/LICENSE-2.0.html).

The deeplink will be built up with the values specified through the wallet API, and currently only supports a request for Signum. In case new possible actions are found to be useful, this protocol can be extended.

The request for Signum deeplink looks like one of the following patterns:

    burst://requestBurst&receiver=[RECEIVER]&amountNQT=[AMOUNTNQT]&feeNQT=[FEENQT]&message=[MESSAGE]&immutable=[ISIMMUTABLE]
    burst://requestBurst&receiver=[RECEIVER]&amountNQT=[AMOUNTNQT]&feeSuggestionType=[FEESUGGESTIONTYPE]&message=[MESSAGE]&immutable=[ISIMMUTABLE]

The meanings of these fields are the following:

* [RECEIVER]: Address that should receive the Signa
* [AMOUNTNQT]: Amount of Signa that should be sent
* [FEENQT]: An exact amount of Signa that should be used as the fee value
* [FEESUGGESTIONTYPE]: A [SIP-13 suggested transaction fee type](sip-3.md)
* [MESSAGE]: An extra message that can be passed along
* [ISIMMUTABLE]: A boolean field deciding whether the requested transaction should be immutable

In both cases the recipient field is always required. The remaining fields, apart from the message field are required in case [ISIMMUTABLE] is true.

[FEESUGGESTIONTYPE] and [FEENQT] are mutually exclusive. In case both values are given, a preference should be given for [FEENQT].

Additionally, an overlay with the Burst logo will be added to the generated QR code, to make it clear it's for Burst.

### HTTP API
The wallet's HTTP API offers a "generateDeepLink" action, which takes the arguments passed into the resulting deeplink as its arguments.

### Mobile wallet
After scanning the QR code, the engrained deeplink enables opening the mobile wallet to a specified screen.

In case all the required values are specified, this screen can lead to the payment screen which displays the payment values like recipient, amount to pay, either exact fee or suggested fee type, and an additional message.

Otherwise it will lead to a screen that will enable missing values to be filled in.

It's also possible for the requester to use a flag that will define whether the values are immutable or not - in case they're immutable, they can't be changed anymore by the Signa sender in the transaction.

### Point of Sales Systems
Point of Sales Systems should do a HTTP call to their local Signum-Node to generate their required QR code. It is recommended to use at least a standard or priority transaction fee. It is also recommended to pass along a message that enables linking the resulting transaction to somewhere in the Point of Sales System's database, to close up this information loop.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
