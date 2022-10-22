---
sip: 44
title: Account Profiles
description: Proposal for standardized Account Profile Information
author: ohager
discussions-to: <URL>
status: Draft
type: Standards Track (Layer 2)
category: SRC
created: 2022-10-20
---

## Abstract

This proposal suggests a JSON standard for account profiles. The standard is applicable to the application layer (layer 2) and does not require a protocol change. The account profiles can be used for primarily visual representative purposes, but also allow to define validation rules for senders, e.g. necessary memos for exchanges. Due to the extension possibility, additional application specific data can be attached to the profile.  


## Motivation

Signum Accounts can have mutable [additional account information](https://europe.signum.network/api-doc/index.html#post-/api-requestType-setAccountInfo). At this stage, this is just unstructured text of up to 1000 bytes. For applications (centralized or decentralized) it would be advantageous if there were a unified structured format so that an account profile can be written and read consistently. Thus it is possible to display avatars in the wallets or the exlorer instead of the generic (but cool) hashicons. In this profile it is possible to store both generic and application-specific public information, which can then be used as desired. Furthermore, such a profile also allows to provide validation information for sending tokens to this account, such as sending memos to exchange accounts or similar.


## Specification

In the following, a structured JSON description format is presented, which can be extended arbitrarily in addition to various basic information. The corresponding JSON schema is attached to the proposal. 

Mind, that this format is entirely optional. It is just a recommendation and users 


> All data is encoded in UTF-8


_Example_:

```json
{
  "vs": 1,
  "tp": "cex",
  "nm": "Bittrex",
  "ds": "World class exchange at your service",
  "av": { "QmbWqxBEKC3P8tqsKc98xmWNzrzDtRLMiMPL8wBuTGsMnR": "image/gif" },
  "bg": { "QmUFc4dyX7TJn5dPxp8CrcDeedoV18owTBUWApYMuF6Koc": "image/jpeg" },
  "hp": "https://bittrex.com",
  "sr": "^[0-9a-fA-F]{24}$",
  "al": "@somealias",
  "xt": "QmUFc4dyX7TJn5dPxp8CrcDeedoV18owTBUWApYMuF6Koc",
  "sc": ["https://twitter.com/bittrex"]
}
```

| Field Name | Required | Full Name        | Value Format | Example                                                               | Rules                                                                                                                     | Description                                                                                                                                                                                                                  |
|------------|----------|------------------|--------------|-----------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| vs         | yes      | Version          | number       | 1                                                                     |                                                                                                                           | An integer number to determine the formats version number                                                                                                                                                                    |
| nm         | yes      | Name             | string       | Clownsk8ter_42 🥳                                                     | /^.{,24}$/                                                                                                                | The accounts name, which can be arbitrary UTF-8 string with at max 24 chars                                                                                                                                                  |
| tp         | no       | Type             | string       | biz                                                                   | /^biz|bot|cex|dex|dev|hum$/                                                                                       | A three character code classifying this accounts type                                                                                                                                                                        |
| ds         | no       | Description      | string       | Funny skater girl with 🤡 mask and ...                                | /^.{,384}$/                                                                                                               | A more extensive description , which can be arbitrary UTF-8 string with at max 384 chars                                                                                                                                     |
| av         | no       | Avatar           | object       | { "QmbWqxBEKC3P8tqsKc98xmWNzrzDtRLMiMPL8wBuTGsMnR": "image/gif" }     | A dynamic JSON object, which one and only fields key is an IPFS hash (CID0 or CID1), and a valid image Mime Type as value | The profile image aka avatar, stored on IPFS. There is no technical limit applied, but it's a good practice to have small quadratic sized images, e.g. up to 128 KiB                                                         |
| bg         | no       | Background Image | object       | { "QmbWqxBEKC3P8tqsKc98xmWNzrzDtRLMiMPL8wBuTGsMnR": "image/gif" }     | A dynamic JSON object, which one and only fields key is an IPFS hash (CID0 or CID1), and a valid image Mime Type as value | A background image stored on IPFS.   There is no technical limit applied, but it's a good practice to have optimized banner like images, e.g. up to 512 KiB                                                                  |
| hp         | no       | Homepage         | string       | https://bittrex.com                                                   | URL/CID                                                                                                                   | An (sanitized) URL of at maximum 128 characters pointing to a web presence of that account.                                                                                                                                  |
| sr         | no       | Send Rule        | string       | /^[0-9a-fA-F]{64}$/                                                   | A valid Regex                                                                                                             | A regex that needs to be matched when sending to this account, i.e. a memo for bots or exchanges                                                                                                                             |
| sc         | no       | Social Network   | array        | ["https://twitter.com/sk8terclown_42","https://discord.gg/ZGHgCXy45"] | An array of URLs/CIDs                                                                                                     | A list of at max. three (sanitized) URLs or IPFS CIDs of at maximum 92 characters each                                                                                                                                       |
| al         | no       | Signum Alias     | string       | @myalias                                                              | /^@\w{1,100}$/                                                                                                        | An related alias of the Signum chain                                                                                                                                                                                         |
| xt         | no       | Extension        | string       | QmUFc4dyX7TJn5dPxp8CrcDeedoV18owTBUWApYMuF6Koc                        | A valid IPFS CID                                                                                                          | The CID for extended information. The resulting document does not follow any format restrictions, as it completely use case dependent. Good formats are JSON, but also private information in encrypted formats is possible. |

### Custom In-Object Extensions

The mentioned fields are almost all optional. To not overcomplicate minor custom extensions it is possible to add custom fields to this object, as long as the limit of 1000 bytes is not exceeded. As a recommendation a custom field SHOULD have as first character an `x` and SHOULD be only two characters, e.g. `x1`, or `xa` 

> Keep in mind, that custom fields can only be processed by applications, which support it.

_Examples_


```json
{
  "vs": 1,
  "tp": "smc",
  "nm": "Contract A",
  "ds": "Bla bla",
  "av": { "QmbWqxBEKC3P8tqsKc98xmWNzrzDtRLMiMPL8wBuTGsMnR": "image/gif" },
  "bg": { "QmUFc4dyX7TJn5dPxp8CrcDeedoV18owTBUWApYMuF6Koc": "image/jpeg" },
  "hp": "https://foobar.com",
  "sr": "^[0-9a-fA-F]{24}$",
  "al": "@somealias",
  "x1": "https://github.com/foobar/contracts/blob/main/sources/contract.c",
}
```


Not recommended but still conformant objects:


_No x as first char used_
```json
{
  "vs": 1,
  "tp": "hum",
  "nm": "SignumArt Creator",
  "ds": "Some nice artworks from wy side",
  "av": { "QmbWqxBEKC3P8tqsKc98xmWNzrzDtRLMiMPL8wBuTGsMnR": "image/gif" },
  "bg": { "QmUFc4dyX7TJn5dPxp8CrcDeedoV18owTBUWApYMuF6Koc": "image/jpeg" },
  "tw": "mytwitter"
}
```

_More than two characters_

```json
{
  "vs": 1,
  "tp": "hum",
  "nm": "SignumArt Creator",
  "ds": "Some nice artworks from wy side",
  "av": { "QmbWqxBEKC3P8tqsKc98xmWNzrzDtRLMiMPL8wBuTGsMnR": "image/gif" },
  "bg": { "QmUFc4dyX7TJn5dPxp8CrcDeedoV18owTBUWApYMuF6Koc": "image/jpeg" },
  "x-tw": "mytwitter"
}
```


### Payload Size Limit

The JSON object SHOULD be [minimized](https://codebeautify.org/jsonminifier) to reduce the payload.
It should be noted that the total size of 1000 (one thousand) bytes MUST NOT be exceeded. It is the responsibility of the user to pay attention to the total size. By the fact that many fields are optional, a margin is created.
Mind that Unicode characters can occupy more than a single byte. 

#### Compression

> Please review this part and comment here - Personally, I (ohager) do not favor compression.

By using compression, i.e. Zip Deflate (as also used in encrypted P2P messages), and base64 encoding it is possible to reduce the payload even more. 

The drawback of this method is that the description is not human readable anymore (less transparency), and requires additional processing from readers/writers.

### Field `tp`  - Type  

> Please review this part and comment here - I (ohager) thought it might be interesting to add these types

An account CAN be categorized using a three letter code. Following pre-defined codes are suggested:

- Human Account: `hum`
- Smart Contract Account: `smc`  (as description field. The use of `al` is highly recommended)
- Non Smart contract, but automated Account: `bot` 
- Business Account: `biz`
- Centralized Exchange Account: `cex`
- Decentralized Exchange Account: `dex` 
- Other: `oth` 


### Field - `sr`  - Send Rule

The optional `sr` field is a regular expression that MUST be considered by applications, once it is present. The regular expression defines how the senders attached message MUST be formatted. If the rule is violated the sending application MUST NOT send the transaction to this account.

### Field - `al`  - Alias

This optional field relates the account with an alias. The Signum Alias system allows to be mutable, while still on-chain. This way it is possible to even make mutable descriptions for smart contracts, if using this SRC for Smart Contract description fields.


### Field - `xt`  - Extension

The profile information can be extended by any data by means of the `xt` field. The only condition is that they are available via IPFS. In this way any application specific data - also in encrypted form - can be coupled to a profile. This SRC does not prescribe whether and how this data should be structured.


### Image MIME-Types

For the `av` and `bg` field an additional information of the [images MIME-Type](https://mimetype.io/all-types/#image) is required. At least supported image types SHOULD be:

- `image/jpeg`
- `image/png`
- `image/webp`
- `image/gif`
- `image/svg+xml`


### URIs

URIs SHOULD always be sanitized to avoid malicious URIs. In case of shorened URIs, the consuming application MUST run sanitization. This SRC does not prevent malicious URIs. If possible use CIDs instead.

### IPFS Hashes (CIDs)

Images, and extended Metadata are storable on IPFS and can be [addressed using CID](https://docs.ipfs.tech/concepts/content-addressing/). Depending on the version they consist of either 46 characters for CIDv0 or dynamic slighty larger sizes for CIDv1. As recommended by IPFS team, CID v1 should be preferred, as it will turn default soon. 


```
- CID v0: QmbWqxBEKC3P8tqsKc98xmWNzrzDtRLMiMPL8wBuTGsMnR
- CID v1: bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi
```

## Additional Consideration

Although, Smart Contracts do not have separated Account Information like common accounts have, this SRC can be applied as description for Smart Contracts also. This way, interaction with smart contracts could have upfront sending validation also. 

### Alias Forwarding

The combination of the `al` field, even allows Smart Contracts to have _mutable_ information attached to it. It's entirely feasible, that the contract has its _immutable_ description according to this specification and links further _mutable_ information via an alias to it.


## Backwards Compatibility
This proposal is backwards compatible. If a profile does not provide such information further processing of those is skipped and has no impact on either protocol or similar. This SRC is intended for Application Layer (Layer 2) only.


## Reference Implementation

A [JSON Schema](https://json-schema.org/) will be provides and also reference implementation for Data Creation/Validation at least for Javascript will be provided. 

## Security Considerations

It is implicit that all relevant profile metadata is unencrypted and publicly available. The use of profile information is optional (opt-in). Users have to be aware of this and MUST NOT store sensitive data in the metadata. Nevertheless, it is possible to store sensible information in the data referenced by `xt`.

As pointed out, this specification can be used in _immutable_ descriptions of smart contracts, _mutable_ account info and _mutable_ alias data. When resolving the `al` field, it MUST NOT be resolved recursively, i.e. chaining `al` resolving is not allowed.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
