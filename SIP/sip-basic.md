---
sip: basic
title: SIP Purpose and Guidelines
status: Living
type: Informational
author: jjos, frank_the_tank, ohager
created: 2022-01-28
---

## What is an SIP?

SIP stands for Signum Improvement Proposal. A SIP is a design document providing information to the Signum community, or describing a new feature for Signum or its processes or environment. The SIP should provide a concise technical specification of the feature and a rationale for the feature. The SIP author is responsible for building consensus within the community and documenting dissenting opinions.

## SIP Rationale

We intend SIPs to be the primary mechanisms for proposing new features, for collecting community technical input on an issue, and for documenting the design decisions that have gone into Signum. Because the SIPs are maintained as text files in a versioned repository, their revision history is the historical record of the feature proposal.

For Signum implementers, SIPs are a convenient way to track the progress of their implementation. Ideally each implementation maintainer would list the SIPs that they have implemented. This will give end users a convenient way to know the current status of a given implementation or library.

## SIP Types

There are three types of SIP:

- A **Standards Track SIP** describes any change that affects most or all Signum implementations, such as—a change to the network protocol, a change in block or transaction validity rules, proposed application standards/conventions, or any change or addition that affects the interoperability of applications using Signum. Standards Track SIPs consist of three parts—a design document, an implementatio. Furthermore, Standards Track SIPs can be broken down into the following categories:
- 
  - **Core**: improvements requiring a consensus fork
  - **Interface**: includes improvements around client API/RPC. 
  - **SRC**: application-level standards and conventions, including smart contract standards.

- An **Informational SIP** describes an Signum design issue, or provides general guidelines or information to the Signum community, but does not propose a new feature. Informational SIPs do not necessarily represent Signu, community consensus or a recommendation, so users and implementers are free to ignore Informational SIPs or follow their advice.

It is highly recommended that a single SIP contain a single key proposal or new idea. The more focused the SIP, the more successful it tends to be. A change to one client doesn't require an SIP; a change that affects multiple clients, or defines a standard for multiple apps to use, does.

An SIP must meet certain minimum criteria. It must be a clear and complete description of the proposed enhancement. The enhancement must represent a net improvement. The proposed implementation, if applicable, must be solid and must not complicate the protocol unduly.

### Special requirements for Core SIPs

If a **Core** SIP mentions or proposes changes to the SVM (Signum Virtual Machine), it should refer to the instructions by their mnemonics and define the opcodes of those mnemonics at least once. A preferred way is the following:
```
REVERT (0xfe)
```

## SIP Work Flow

### Shepherding an SIP

Before you begin writing a formal SIP, you should vet your idea. Ask the Signum community first if an idea is original to avoid wasting time on something that will be rejected based on prior research. It is thus recommended to open a discussion thread on Signum disccord too.

Once the idea has been vetted, your next responsibility will be to present (by means of an SIP) the idea to the reviewers and all interested parties, invite editors, developers, and the community to give feedback on the aforementioned channels. You should try and gauge whether the interest in your SIP is commensurate with both the work involved in implementing it and how many parties will have to conform to it. For example, the work required for implementing a Core SIP will be much greater than for an SRC and the SIP will need sufficient interest from the Signum client teams. Negative community feedback will be taken into consideration and may prevent your SIP from moving past the Draft stage.

### Core SIPs

For Core SIPs, given that they require client implementations to be considered **Final** (see "SIPs Process" below), you will need to either provide an implementation for clients or convince clients to implement your SIP. 


### SIP Process 

The following is the standardization process for all SIPs in all tracks:


**Draft** - The first formally tracked stage of an SIP in development. An SIP is merged by an SIP Editor into the SIP repository when properly formatted.

**Review** - An SIP Author marks an SIP as ready for and requesting Peer Review.

**Last Call** - This is the final review window for an SIP before moving to `Final`. An SIP editor will assign `Last Call` status and set a review end date (`last-call-deadline`), typically 14 days later.

If this period results in necessary normative changes it will revert the SIP to `Review`.

**Final** - This SIP represents the final standard. A Final SIP exists in a state of finality and should only be updated to correct errata and add non-normative clarifications.

**Stagnant** - Any SIP in `Draft` or `Review` or `Last Call` if inactive for a period of 6 months or greater is moved to `Stagnant`. An SIP may be resurrected from this state by Authors or SIP Editors through moving it back to `Draft` or it's earlier status. If not resurrected, a proposal may stay forever in this status. 

**Withdrawn** - The SIP Author(s) have withdrawn the proposed SIP. This state has finality and can no longer be resurrected using this SIP number. If the idea is pursued at later date it is considered a new proposal.

**Living** - A special status for SIPs that are designed to be continually updated and not reach a state of finality. This includes most notably SIP-Basic.

## What belongs in a successful SIP?

Each SIP should have the following parts:

- Preamble - A RFC822 like header containing metadata about the SIP, including the SIP number, a short descriptive title (limited to a maximum of 44 characters), a description (limited to a maximum of 140 characters), and the author details. Irrespective of the category, the title and description should not include SIP number. See [below](./sip-basic.md#sip-header-preamble) for details.
- Abstract - Abstract is a multi-sentence (short paragraph) technical summary. This should be a very terse and human-readable version of the specification section. Someone should be able to read only the abstract to get the gist of what this specification does.
- Motivation (*optional) - A motivation section is critical for SIPs that want to change the Ethereum protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the SIP solves. SIP submissions without sufficient motivation may be rejected outright.
- Specification - The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Signum platforms .
- Rationale - The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.
- Backwards Compatibility - All SIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The SIP must explain how the author proposes to deal with these incompatibilities. SIP submissions without a sufficient backwards compatibility treatise may be rejected outright.
- Test Cases - Test cases for an implementation are mandatory for SIPs that are affecting consensus changes. Tests should either be inlined in the SIP as data (such as input/expected output pairs, or included in `../assets/sip-###/<filename>`.
- Reference Implementation - An optional section that contains a reference/example implementation that people can use to assist in understanding or implementing this specification.
- Security Considerations - All SIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life-cycle of the proposal. E.g. include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. SIP submissions missing the "Security Considerations" section will be rejected. An SIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.
- Copyright Waiver - All SIPs must be in the public domain. See the bottom of this SIP for an example copyright waiver.

## SIP Formats and Templates

SIPs should be written in [markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) format. There is a [template](https://github.com/ethereum/EIPs/blob/master/eip-template.md) to follow.

## SIP Header Preamble

Each SIP must begin with an [RFC 822](https://www.ietf.org/rfc/rfc822.txt) style header preamble, preceded and followed by three hyphens (`---`). This header is also termed ["front matter" by Jekyll](https://jekyllrb.com/docs/front-matter/). The headers must appear in the following order.

`eip`: *SIP number* (this is determined by the SIP editor)

`title`: *The SIP title is a few words, not a complete sentence*

`description`: *Description is one full (short) sentence*

`author`: *The list of the author's or authors' name(s) and/or username(s), or name(s) and email(s). Details are below.*

`discussions-to`: *The url pointing to the official discussion thread*

`status`: *Draft, Review, Last Call, Final, Stagnant, Withdrawn, Living*

`last-call-deadline`: *The date last call period ends on* (Optional field, only needed when status is `Last Call`)

`type`: *One of `Standards Track`, `Meta`, or `Informational`*

`category`: *One of `Core`, `Networking`, `Interface`, or `ERC`* (Optional field, only needed for `Standards Track` EIPs)

`created`: *Date the SIP was created on*

`requires`: *SIP number(s)* (Optional field)

`withdrawal-reason`: *A sentence explaining why the SIP was withdrawn.* (Optional field, only needed when status is `Withdrawn`)

Headers that permit lists must separate elements with commas.

Headers requiring dates will always do so in the format of ISO 8601 (yyyy-mm-dd).

#### `author` header

The `author` header lists the names, email addresses or usernames of the authors/owners of the SIP. Those who prefer anonymity may use a username only, or a first name and a username. The format of the `author` header value must be:

> Random J. User &lt;address@dom.ain&gt;

or

> Random J. User (@username)

if the email address or GitHub username is included, and

> Random J. User

if the email address is not given.

It is not possible to use both an email and a GitHub username at the same time. If important to include both, one could include their name twice, once with the GitHub username, and once with the email.

At least one author must use a GitHub username, in order to get notified on change requests and have the capability to approve or reject them.

#### `discussions-to` header

While an SIP is a draft, a `discussions-to` header will indicate the URL where the SIP is being discussed.


#### `type` header

The `type` header specifies the type of SIP: Standards Track, Meta, or Informational. If the track is Standards please include the subcategory (core, networking, interface, or SRC).

#### `category` header

The `category` header specifies the SIP's category. This is required for standards-track SIPs only.

#### `created` header

The `created` header records the date that the SIP was assigned a number. Both headers should be in yyyy-mm-dd format, e.g. 2001-08-14.

#### `requires` header

SIPs may have a `requires` header, indicating the EIP numbers that this EIP depends on.

## Linking to other SIPs

References to other SIPs should follow the format `SIP-N` where `N` is the SIP number you are referring to.  Each SIP that is referenced in an SIP **MUST** be accompanied by a relative markdown link the first time it is referenced, and **MAY** be accompanied by a link on subsequent references.  The link **MUST** always be done via relative paths so that the links work in this GitHub repository, forks of this repository, the main SIPs site, mirrors of the main SIP site, etc.  For example, you would link to this SIP with `[SIP-1](./sip-1.md)`.

## Auxiliary Files

Images, diagrams and auxiliary files should be included in a subdirectory of the `assets` folder for that SIP as follows: `assets/sip-N` (where **N** is to be replaced with the EIP number). When linking to an image in the SIP, use relative links such as `../assets/sip-1/image.png`.



## SIP Author Responsibilities

For each new SIP that comes in, an author does the following:

- Read the SIP to check if it is ready: sound and complete. The ideas must make technical sense, even if they don't seem likely to get to final status.
- The title should accurately describe the content.
- Check the SIP for language (spelling, grammar, sentence structure, etc.), markup (GitHub flavored Markdown), code style


## Style Guide

### SIP numbers

When referring to an SIP by number, it should be written in the hyphenated form `SIP-X` where `X` is the EIP's assigned number.

### RFC 2119

SIPs are encouraged to follow [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt) for terminology and to insert the following at the beginning of the Specification section:

> The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in RFC 2119.


## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
