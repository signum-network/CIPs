# Signum Improvement Proposals (SIPs)

Signum Improvement Proposals (SIPs) describe standards for the Signum platform, including core protocol specifications, client APIs, and contract standards.

**Before you initiate a pull request**, please read the [SIP-Basics](./SIP/sip-basic.md) process document.
There is a [template SIP here](./SIP/sip-template.md). 

If you use your email address, that address must be the one publicly shown on [your GitHub profile](https://github.com/settings/profile).

## Project Goal

The Signum Improvement Proposals repository exists as a place to share concrete proposals with potential users of the proposal and the Signum community at large.

## SIP status terms

-   **Idea** - An idea that is pre-draft. This is not tracked within the SIP Repository.
-   **Draft** - The first formally tracked stage of an SIP in development. 
-   **Review** - An SIP Author marks an SIP as ready for and requesting Peer Review.
-   **Last Call** - This is the final review window for an SIP before moving to FINAL. An SIP editor will assign Last Call status and set a review end date (`last-call-deadline`), typically 14 days later. If this period results in necessary normative changes it will revert the SIP to Review.
-   **Final** - This SIP represents the final standard. A Final SIP exists in a state of finality and should only be updated to correct errata and add non-normative clarifications.
-   **Stagnant** - Any SIP in Draft or Review if inactive for a period of 6 months or greater is moved to Stagnant. An EIP may be resurrected from this state by Authors or EIP Editors through moving it back to Draft.
-   **Withdrawn** - The SIP Author(s) have withdrawn the proposed EIP. This state has finality and can no longer be resurrected using this SIP number. If the idea is pursued at later date it is considered a new proposal.
-   **Living** - A special status for SIPs that are designed to be continually updated and not reach a state of finality. This includes most notably SIP-Basics.

## SIP Types

SIPs are separated into a number of types:

### Standard Track (45)

#### Core (1)

Improvements requiring a consensus fork 

#### Interface (39)

Includes improvements around client API/RPC specifications and standards, and also certain language-level standards like method names and contract ABIs. T

#### SRC (1)

Application-level standards and conventions, including contract standards such as smart contracts  

### Informational (6)

Describes a Signum design issue, or provides general guidelines or information to the Signum community, but does not propose a new feature. Informational SIPs do not necessarily represent Ethereum community consensus or a recommendation, so users and implementers are free to ignore Informational SIPs or follow their advice.

## Signum Capability Improvement Proposals

| Number                              | Layer                 | Title                                                 | Owner         | Type          | Status   |
| ----------------------------------- | --------------------- | ----------------------------------------------------- | ------------- | ------------- | -------- |
| [1](cip-0001.md "wikilink")  | Applications          | Dynamic BRS Node Capabilities                         | @rico666      | Informational | Active   |
| [2](cip-0002.md "wikilink")  | Consensus (hard fork) | Quadruple Block Size                                  | @rico666      | Standard      | Active   |
| [3](cip-0003.md "wikilink")  | Consensus (hard fork) | Variable Slot-based Fees                              | @rico666      | Standard      | Active   |
| [4](cip-0004.md "wikilink")  | Consensus (hard fork) | Multi-Out Transactions                                | @rico666      | Standard      | Active   |
| [5](cip-0005.md "wikilink")  | Consensus (hard fork) | POC 2                                                 | @Quibus       | Standard      | Active   |
| [6](cip-0006.md "wikilink")  | Enhancement           | Unconfirmed Tx Queue Optimizations                    | @Brabantian   | Standard      | Active   |
| [7](cip-0007.md "wikilink")  | Enhancement           | Differential Unconfirmed Tx Propagation               | @Brabantian   | Standard      | Active   |
| [8](cip-0008.md "wikilink")  | Informal              | Define Tx and Balance "Dust"                          | @rico666      | Standard      | Draft    |
| [9](cip-0009.md "wikilink")         | Consensus (hard fork) | To-All Transactions                                   | @rico666      | Standard      | Draft    |
| [10](cip-0010.md "wikilink") | Consensus (hard fork) | Anchor Real-World Data in Blockchain                  | @rico666      | Standard      | Draft    |
| [11](cip-0011.md "wikilink")        | Consensus (hard fork) | Tethered Assets                                       | @rico666      | Standard      | Draft    |
| [12](cip-0012.md "wikilink") | Enhancement           | BFS - Burst File System                               | @JohnnyDeluxe | Standard      | Draft    |
| [13](cip-0013.md "wikilink") | Enhancement           | Suggested Transaction Fees                            | @Brabantian   | Standard      | Active   |
| [14](cip-0014.md "wikilink")        | Enhancement           | Burst Actions through Deeplink QR-codes               | @Brabantian   | Standard      | Active   |
| [15](cip-0015.md "wikilink")        | Consensus (hard fork) | RWFDS-enabled FEE\_QUANT Introspection and Adjustment | @Frank72      | Standard      | Draft    |
| [16](cip-0016.md "wikilink")        | Enhancement           | PoC2.X16 - A New Optimized Plot File Format           | @JohnnyFFM    | Standard      | Draft    |
| [17](cip-0017.md "wikilink")        | Enhancement           | Differential UT Propagation in push & pull            | @Brabantian   | Standard      | Active |
| [18](cip-0018.md "wikilink")        | Applications           | Cross-Platform Wallet UI            | @blankey1337, @ohager   | Standard      | Active |
| [19](cip-0019.md "wikilink")        | Enhancement           | Multi-Out: View Incoming Transactions            | @harry1453   | Standard      | Active |
| [20](cip-0020.md "wikilink")        | Enhancement (Hard Fork)          | Updated AT fees            | @frank_the_tank, @jjos, @burstjack   | Standard      | Active |
| [21](cip-0021.md "wikilink")        | Enhancement (Hard Fork) | Adjustment for Asset-Issuance fee | @frank_the_tank, @burstjack   | Standard      | Active |
| [22](cip-0022.md "wikilink")        | Enhancement          | Burst Deep Link Specification  | @ohager   | Standard      | Active |
| [23](cip-0023.md "wikilink")        | Enhancement (Hard Fork) | Enforce slot fees  | @CurbShifter   | Standard      | Active |
| [24](cip-0024.md "wikilink")        | Enhancement (Hard Fork) | New deadline algorithm based on a logarithm transformation  | @jjos   | Standard      | Active |
| [25](cip-0025.md "wikilink")        | Enhancement (Hard Fork) | Minor Network Changes | @harry1453  | Standard      | Active |
| [26](cip-0026.md "wikilink")        | Enhancement | Extended Reed Solomon Address Format | @ohager, jjos  | Standard      | Active |
| [27](cip-0027.md "wikilink")        | Enhancement (Hard Fork) | Proof of Commitment (PoC+) consensus | @jjos   | Standard      | Active |
| [28](cip-0028.md "wikilink")        | Enhancement (Hard Fork) | Updated AT max steps and AT API | @frank_the_tank, @jjos, @ohager   | Standard      | Active |
| [29](cip-0029.md "wikilink")        | Enhancement (Hard Fork) | Mininum mining incentives | @frank_the_tank, @jjos, @shefas   | Standard      | Active |
| [30](cip-0030.md "wikilink")        | Enhancement (Hard Fork) | Carbon Contracts | @frank_the_tank, @jjos   | Standard      | Active |
| [31](cip-0031.md "wikilink")        | Enhancement (Hard Fork) | New fee regime |  @jjos, @frank_the_tank   | Standard      | Active |
| [32](cip-0032.md "wikilink")        | Enhancement (Hard Fork) | PoC+ Polishing |  @jjos, @ohager, @frank_the_tank   | Standard      | Active |
| [33](cip-0033.md "wikilink")        | Enhancement (Hard Fork) | Smart Tokens|  @jjos, @frank_the_tank   | Standard      | Draft |


<!-- IMPORTANT!  See the instructions at the top of this page, do NOT JUST add CIPs here! -->
