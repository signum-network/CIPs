---
sip: 44
title: Profiles
description: Proposal for standardized Profile Information
author: ohager
status: Review
Last Call: 2022-11-22
type: Standard
category: SRC
created: 2022-10-20
---

[Established SIP44](../../SIP/sip-0044.md) - See details there.

## Abstract

This proposal suggests a JSON standard for account profiles. The standard is applicable to the application layer (layer 2) and does not require a protocol change. The account profiles can be used for primarily visual representative purposes, but also allow to define validation rules for senders, e.g. necessary memos for exchanges. Due to the extension possibility, additional application specific data can be attached to the profile.  


## Compliance

According to the established specifiction a [JSON Schema](./src44-json-schema.json) can be used to check profile data against the standard. Using a [JSON Schema Validator](https://www.jsonschemavalidator.net/) helps to check data against the standard


Additionally, a SRC44 conformant implementation is available in the [SignumJS SDK](https://github.com/signum-network/signumjs/tree/main/packages/standards/src/src44)



## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
