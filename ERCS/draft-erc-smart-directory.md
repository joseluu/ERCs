---
eip: TBD
title: Directory of certified smart-contracts
status: Draft
type: Standards Track
description: Allows a recognized entity to list the valid smart-contract addresses of the actors in its ecosystem.
author: Jose Luu <jose.luu@bpce.fr>, Cyril Vignet <cyril.vignet@bpce.fr>, Vincent Griffault <vincent.griffault@cera.caisse-epargne.fr>, Frederic Faure <frederic.faure@banque-france.fr>, Clement Delaneau <clement.delaneau@banque-france.fr>
created: 2025-9-2
---

## Preliminary note

The ERC presented here originates from a project named "**SmartDirectory**", for convenience in the draft redaction the term "SmartDirectory" is used to designate the smart-contract model subject of this ERC.

## Abstract

The SmartDirectory is an **administered blockchain whitelist** that addresses the proliferation of addresses by ensuring their authenticity for important transactions. It allows an organisation, called a **registrant**, to list the valid smart contract addresses it has deployed. Once an administrator of the recognized authority approves a registrant, that registrant can then record their service-related smart contract addresses in the **"references" list**. Overall the "SmartDirectory" facilitates on-chain verification and the identification and management of smart contract ecosystems.

## Motivation

The rapid proliferation of smart contract addresses poses a critical challenge to users and other smart contracts, necessitating robust mechanisms for **authenticity verification** for any transactions using them. The SmartDirectory emerges as an essential **administered blockchain whitelist**, directly addressing this issue by providing a structured solution for managing trust on-chain.

Its core purpose is to enable organisations, known as **registrants**, to securely expose and maintain the valid smart contract addresses that they operate. Through a streamlined process, administrators of a regognized authority approve registrants, who then gain the ability to deploy and record their service-related smart contracts in a dedicated **"references" list**. The SmartDirectory is vital for enhancing security, transparency, and operational efficiency in an increasingly complex world. For newcommers as well as for seasonned users, it greatly facilitates and brings certainty to the "do your homework" address validation phase.

In terms of automation, the directory allows **on-chain verification** by allowing:
* smart wallets to check and validate the addresses upon usage
* other smart contracts to perform crucial addresses checks
* update dynamically the addresses list to be checked 

## Specification

The following interface and rules are normative. The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Definitions

* **recognized authority**
  - An organisation which is well known by potential users who decides to deploy a SmartDirectory contract subject to this ERC. Such organisations can be major token issuers, exchanges, state regulators, non profit foundations, industry consortia ...
* **SmartDirectory**
  - An administered blockchain whitelist smart-contract that serves as a shared, on-chain repository for lists of addresses, which can be either Externally Owned Accounts (EOA) or smart contracts. 
  - It functions as a decentralized directory composed of references (smart contract addresses) and their issuers (registrants). 
* **Registrant**
  - A service provider who, after receiving approval from an administrator of the recognized authority, is registered by that administrator in the SmartDirectory.
  - The registrant must have blockchain access and signing capability in order to write address records into the SmartDirectory to build its references list. 
  - Once registered, a registrant can deploy smart contracts for the purpose of their services and record their addresses in the references list. 
  - The registrant must provide and keep updated its own URI for its clients to consult service terms and possibly register for services. 
  - The registrant's address can be an EOA or another smart contract. 
* **Reference**
  - A smart contract addresse issued by a registrant. 
  - The core data is held within the SmartDirectory's "references table," which contains all declared smart contract addresses; these can also be EOAs. 
  - Each reference includes: the registrant's address, the reference's address, a project ID, the reference type, the reference version, and a status.
* **Administrator**
  - The SmartDirectory contract is operated from one or several administrator addresses.
  - In its simple form the SmartDirectory contract administrator address is the contract deployer address usually called the "owner"
  - Provision is made to allow several distinct administrator addresses (see Optional Features)
  - Administrators have the authority to add or invalidate registrants on the "registrants list".
  - Administrators addresses are responsible for (de)activating, configuring, and managing the decentralized directory. 
* **ReferenceStatus**
  - A component within the statusHistory of a reference, tracking its evolution over time. 
  - It contains a status (a free-text string describing the current state, e.g., "in production," "pending," "abandoned") and a timeStamp (the exact time of the status update). 
- **SmartTokens**
  - A concept where a token (fungible or non-fungible) is programmed to consult a SmartDirectory to filter addresses for transactions (e.g., transfers, minting). 
  - This allows for access control mechanisms within token ecosystems; if the list of references in the SmartDirectory changes, the SmartToken does not need to be modified.
  - They are configured with smart\_directory and registrant\_address variables to enable this consultation functionality. 
* **URI**
  - A string that a registrant can provide and update to offer additional information about their services, accessible to clients (Web2 entry point). 
  - The SmartDirectory itself can have a contractUri at deployment, describing the contract. 


### Interface
#### constructor, creation and setter APIs
##### constructor parameters
- _contractUri (string):
    A non-modifiable string URI that describes the SmartDirectory contract itself. 
    This URI allows the recognized authority to provide descriptive information about itself to the users.

#####      setActivationCode(uint256 _activationCode)
 _activation code values: 
-  0 notActivated (initial value at deployment time)
-  1 activated
-  2 endOfLife
It is important to signal to the users that a SmartDirectory has reached end of life so that they don't inadvertently rely on outdated information

##### createRegistrant(address registrantAddress)
 Creates a new registrant. This can only be called by one of the administrators.

##### disableRegistrant(address registrantAddress)
 Disables a registrant by setting its status to invalid, preventing them from creating new references. 
 This can only be called by one of the administrators
 Once disabled, the registrant cannot not be re-enabled unless the optional registrant status audit trail is implemented
 
##### createReference(address referenceAddress, string projectId, string referenceType, string referenceVersion, string status)
 Creates a new reference by a registrant giving an initial status. 
 Registrant must have been created by the administrator, the msg.sender is implicitly used as registrantAddress
 The registrant must not be disabled for the reference to be created

##### updateRegistrantUri(string registrantUri):
 Allows a registrant (msg.sender) to update their registrant_uri.

 #####     updateReferenceStatus(address referenceAddress, string newStatus)
 Adds a new status and timestamp to a reference's statusHistory

#### information API (getters)

##### getReferenceStatus(address referenceAddress)
 Returns the latest status and timestamp of a reference
 This is the main information entry for the public

##### getReference(address referenceAddress)
 Returns all the informations known about a reference:
  -       address registrantAddress,
  -      uint256 registrantIndex,
 -       string memory projectId,
  -      string memory referenceType,
  -      string memory referenceVersion,
  -      string memory status,

Ma proposition serait de modifier "projectID" en "referenceDescription". Le rationel est que refernceDescription comporte un JSON qui puisse inclure referenceTtitel, referenceMetadata, codeHash, ABI, referenceDocumentation (proposés par BdF) plus tout autre balise que le regsitrant peut vouloir. 
Les éléments isolés (referenceType, referenceVersion) sont les éléments qui peuvent être vérifiés par un smartContract. l'idée est d'éviter que le smartContract fasse du parsinG

##### getContractUri() String
 Returns the URI given at contract deployment time
 This URI informs the user of the identity of the recognized authority managing the contract
 see also: **Security Considerations**

#####      getActivationCode()


#### Status values 
TBD

##### Contract Activation code

##### Registrant status

##### Reference status

###   Required Behavior
TBD
###   Optional Features
####  distinct supplementary administrator addresses
 This feature allows the deployer to give adminstration power to other addresses specified at deployement time
 This may help if the organization of the recognized authority requires such separation
 In this case, the constructor needs receive the administration addresses as parameters:
- _parentAddress1 (address):
    ◦ The address of the first SmartDirectory administrator.
    ◦ One of two addresses designated as creators/administrators, with rights to add or invalidate registrants.
    ◦ Must be different from _parentAddress2 and not address(0).
- _parentAddress2 (address):
    ◦ The address of the second SmartDirectory administrator.
    ◦ Similar to _parentAddress1, it has administrative rights.
    ◦ Must be different from _parentAddress1 and not address(0).
####    consultable audit trail for the reference statuses
 This feature allows recording and exposing to the user all the past status changes of a reference
#####     getReference(address referenceAddress) see full description above
returns an additional information: the timestamp of the status
#####     getReferenceLastStatusIndex(address referenceAddress)
 In the optional case where an audit trail of the status changes is recorded
 This allows to retrieve all the changes by iterating **getReferenceStatusAtIndex**
#####     getReferenceStatusAtIndex(address referenceAddress, uint256 statusIndex)
 Returns the status and timestamp at a specific index in the statusHistory


####    registrant status management
##### isValidRegistrant (_registrantAddress)
 An implementation may want to distinguish between non existent registrants and invalid registrants
 This returns wether the registrant is disabled, a non existent registrant will raise an error
##### registrant status audit trail
 In the optional case where an audit trail of the registrant status changes is recorded
 This feature is used if registrant drop out of compliance and needs to be reenacted later

#### admincode for open/closed management of the contract ?
#### getContractVersion
  This feature allows to track code versions of the contract
#### fonctions for enumerating the contents
##### getDisabledRegistrants() address[]
 Returns an address table listing all the registrants that are disabled
##### getRegistrantLastIndex()
 Returns the last index used in the registrants list
 This allows to retrieve all the registrants
##### getRegistrantAtIndex(uint256 index)
 Returns the address and URI of a declarant at a specific index
##### getReferencesList(address registrantAddress)
 Returns an array of references for a given declarant


## Rationale
This contract ERC offers a solution to the growing complexity of blockchain interactions by providing a standardized, on-chain, and dynamically verifiable mechanism for managing trusted addresses and smart contracts, thereby enhancing security, flexibility, and operational efficiency within decentralized applications and tokenized economies.
### Administration Considerations
The deploying recognized authority needs to have an adminstrative off-chain process for organizations to apply and be vetted as registrants, as well as maintain the vetted registrant status. The registrant list must be updated acccordingly, possibly disabling registrants at some point when the no longer match the vetting requirements.

Each registrant organization is sole responsible of its own references (contract addresses), this includes
keeping each URI alive with user oriented information, keeping the version number and status updated to reflect the state of its publicly reachable contracts addresses.

### Parameter Selection and Degenerate Cases
### Considered Alternatives

## Reference Implementation
TBD

## Security Considerations
### URI cross references
#### recognized authority URI
When consulting the recognized authority URI, the data served should contain the address of the SmartDirectory contract as meta-data in machine readable format and possibly in human readable format. The purpose of this cross reference is to avoid a rogue contract claiming having been issued by a recognized authority.
example of cross reference data:
X-Blockchain-Addresses: {"1": "0x1234567890abcdef1234567890abcdef12345678", "56": "0xabcdef1234567890abcdef1234567890abcdef12"}
If the recognized authority has deployed the SmartDirectory contract at the same address on all blockchains the "*" number must be used instead of an actual blockchain number.
In order to prevent alteration, the URI is supplied at contract deployment time to the solidity constructor and cannot be changed later.
#### registrant URI
When consulting the registrant URI, the data served at the URI should contain the address of the registrant as meta-data such as X-Blockchain-Address: "0x1234567890abcdef1234567890abcdef12345678"


## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).


## Help and documentation for filling te ERC

- Preamble - RFC 822 style headers containing metadata about the EIP, including the EIP number, a short descriptive title (limited to a maximum of 44 characters), a description (limited to a maximum of 140 characters), and the author details. Irrespective of the category, the title and description should not include EIP number. See [below](./eip-1.md#eip-header-preamble) for details.
- Abstract - Abstract is a multi-sentence (short paragraph) technical summary. This should be a very terse and human-readable version of the specification section. Someone should be able to read only the abstract to get the gist of what this specification does.
- Motivation *(optional)* - A motivation section is critical for EIPs that want to change the Ethereum protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the EIP solves. This section may be omitted if the motivation is evident.
- Specification - The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Ethereum platforms (besu, erigon, ethereumjs, go-ethereum, nethermind, or others).
- Rationale - The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale should discuss important objections or concerns raised during discussion around the EIP.
- Backwards Compatibility *(optional)* - All EIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their consequences. The EIP must explain how the author proposes to deal with these incompatibilities. This section may be omitted if the proposal does not introduce any backwards incompatibilities, but this section must be included if backward incompatibilities exist.
- Test Cases *(optional)* - Test cases for an implementation are mandatory for EIPs that are affecting consensus changes. Tests should either be inlined in the EIP as data (such as input/expected output pairs) or included in `../assets/eip-###/<filename>`. This section may be omitted for non-Core proposals.
- Reference Implementation *(optional)* - An optional section that contains a reference/example implementation that people can use to assist in understanding or implementing this specification. This section may be omitted for all EIPs.
- Security Considerations - All EIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life-cycle of the proposal. E.g. include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. EIP submissions missing the "Security Considerations" section will be rejected. An EIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.
- Copyright Waiver - All EIPs must be in the public domain. The copyright waiver MUST link to the license file and use the following wording: `Copyright and related rights waived via [CC0](../LICENSE.md).`

## EIP Formats and Templates

EIPs should be written in [markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) format. There is a [template](https://github.com/ethereum/EIPs/blob/master/eip-template.md) to follow.

## EIP Header Preamble

Each EIP must begin with an [RFC 822](https://www.ietf.org/rfc/rfc822.txt) style header preamble, preceded and followed by three hyphens (`---`). This header is also termed ["front matter" by Jekyll](https://jekyllrb.com/docs/front-matter/). The headers must appear in the following order.

`eip`: *EIP number*

`title`: *The EIP title is a few words, not a complete sentence*

`description`: *Description is one full (short) sentence*

`author`: *The list of the author's or authors' name(s) and/or username(s), or name(s) and email(s). Details are below.*

`discussions-to`: *The url pointing to the official discussion thread*

`status`: *Draft, Review, Last Call, Final, Stagnant, Withdrawn, Living*

`last-call-deadline`: *The date last call period ends on* (Optional field, only needed when status is `Last Call`)

`type`: *One of `Standards Track`, `Meta`, or `Informational`*

`category`: *One of `Core`, `Networking`, `Interface`, or `ERC`* (Optional field, only needed for `Standards Track` EIPs)

`created`: *Date the EIP was created on*

`requires`: *EIP number(s)* (Optional field)

`withdrawal-reason`: *A sentence explaining why the EIP was withdrawn.* (Optional field, only needed when status is `Withdrawn`)

Headers that permit lists must separate elements with commas.

Headers requiring dates will always do so in the format of ISO 8601 (yyyy-mm-dd).

### `author` header

The `author` header lists the names, email addresses or usernames of the authors/owners of the EIP. Those who prefer anonymity may use a username only, or a first name and a username. The format of the `author` header value must be:

> Random J. User &lt;address@dom.ain&gt;

or

> Random J. User (@username)

or

> Random J. User (@username) &lt;address@dom.ain&gt;

if the email address and/or GitHub username is included, and

> Random J. User

if neither the email address nor the GitHub username are given.

At least one author must use a GitHub username, in order to get notified on change requests and have the capability to approve or reject them.

### `discussions-to` header

While an EIP is a draft, a `discussions-to` header will indicate the URL where the EIP is being discussed.

The preferred discussion URL is a topic on [Ethereum Magicians](https://ethereum-magicians.org/). The URL cannot point to Github pull requests, any URL which is ephemeral, and any URL which can get locked over time (i.e. Reddit topics).

### `type` header

The `type` header specifies the type of EIP: Standards Track, Meta, or Informational. If the track is Standards please include the subcategory (core, networking, interface, or ERC).

### `category` header

The `category` header specifies the EIP's category. This is required for standards-track EIPs only.

### `created` header

The `created` header records the date that the EIP was assigned a number. Both headers should be in yyyy-mm-dd format, e.g. 2001-08-14.

### `requires` header

EIPs may have a `requires` header, indicating the EIP numbers that this EIP depends on. If such a dependency exists, this field is required.

A `requires` dependency is created when the current EIP cannot be understood or implemented without a concept or technical element from another EIP. Merely mentioning another EIP does not necessarily create such a dependency.

## Linking to External Resources

Other than the specific exceptions listed below, links to external resources **SHOULD NOT** be included. External resources may disappear, move, or change unexpectedly.

The process governing permitted external resources is described in [EIP-5757](./eip-5757.md).

### Execution Client Specifications

Links to the Ethereum Execution Client Specifications may be included using normal markdown syntax, such as:

```markdown
[Ethereum Execution Client Specifications](https://github.com/ethereum/execution-specs/blob/9a1f22311f517401fed6c939a159b55600c454af/README.md)
```

Which renders to:

[Ethereum Execution Client Specifications](https://github.com/ethereum/execution-specs/blob/9a1f22311f517401fed6c939a159b55600c454af/README.md)

Permitted Execution Client Specifications URLs must anchor to a specific commit, and so must match this regular expression:

```regex
^(https://github.com/ethereum/execution-specs/(blob|commit)/[0-9a-f]{40}/.*|https://github.com/ethereum/execution-specs/tree/[0-9a-f]{40}/.*)$
```

### Execution Specification Tests

Links to the Ethereum Execution Specification Tests (EEST) may be included using normal markdown syntax, such as:

```markdown
[Ethereum Execution Specification Tests](https://github.com/ethereum/execution-spec-tests/blob/c9b9307ff320c9bb0ecb9a951aeab0da4d9d1684/README.md)
```

Which renders to:

[Ethereum Execution Specification Tests](https://github.com/ethereum/execution-spec-tests/blob/c9b9307ff320c9bb0ecb9a951aeab0da4d9d1684/README.md)

Permitted Execution Specification Tests URLs must anchor to a specific commit, and so must match one of these regular expressions:

```regex
^https://(www\.)?github\.com/ethereum/execution-spec-tests/(blob|tree)/[a-f0-9]{40}/.+$
```

```regex
^https://(www\.)?github\.com/ethereum/execution-spec-tests/commit/[a-f0-9]{40}$
```

### Consensus Layer Specifications

Links to specific commits of files within the Ethereum Consensus Layer Specifications may be included using normal markdown syntax, such as:

```markdown
[Beacon Chain](https://github.com/ethereum/consensus-specs/blob/26695a9fdb747ecbe4f0bb9812fedbc402e5e18c/specs/sharding/beacon-chain.md)
```

Which renders to:

[Beacon Chain](https://github.com/ethereum/consensus-specs/blob/26695a9fdb747ecbe4f0bb9812fedbc402e5e18c/specs/sharding/beacon-chain.md)

Permitted Consensus Layer Specifications URLs must anchor to a specific commit, and so must match this regular expression:

```regex
^https://github.com/ethereum/consensus-specs/(blob|commit)/[0-9a-f]{40}/.*$
```

### Networking Specifications

Links to specific commits of files within the Ethereum Networking Specifications may be included using normal markdown syntax, such as:

```markdown
[Ethereum Wire Protocol](https://github.com/ethereum/devp2p/blob/40ab248bf7e017e83cc9812a4e048446709623e8/caps/eth.md)
```

Which renders as:

[Ethereum Wire Protocol](https://github.com/ethereum/devp2p/blob/40ab248bf7e017e83cc9812a4e048446709623e8/caps/eth.md)

Permitted Networking Specifications URLs must anchor to a specific commit, and so must match this regular expression:

```regex
^https://github.com/ethereum/devp2p/(blob|commit)/[0-9a-f]{40}/.*$
```

### Portal Specifications

Links to specific commits of files within the Ethereum Portal Specifications may be included using normal markdown syntax, such as:

```markdown
[Portal Wire Protocol](https://github.com/ethereum/portal-network-specs/blob/5e321567b67bded7527355be714993c24371de1a/portal-wire-protocol.md)
```

Which renders as:

[Portal Wire Protocol](https://github.com/ethereum/portal-network-specs/blob/5e321567b67bded7527355be714993c24371de1a/portal-wire-protocol.md)

Permitted Networking Specifications URLs must anchor to a specific commit, and so must match this regular expression:

```regex
^https://github.com/ethereum/portal-network-specs/(blob|commit)/[0-9a-f]{40}/.*$
```

### World Wide Web Consortium (W3C)

Links to a W3C "Recommendation" status specification may be included using normal markdown syntax. For example, the following link would be allowed:

```markdown
[Secure Contexts](https://www.w3.org/TR/2021/CRD-secure-contexts-20210918/)
```

Which renders as:

[Secure Contexts](https://www.w3.org/TR/2021/CRD-secure-contexts-20210918/)

Permitted W3C recommendation URLs MUST anchor to a specification in the technical reports namespace with a date, and so MUST match this regular expression:

```regex
^https://www\.w3\.org/TR/[0-9][0-9][0-9][0-9]/.*$
```

### Web Hypertext Application Technology Working Group (WHATWG)

Links to WHATWG specifications may be included using normal markdown syntax, such as:

```markdown
[HTML](https://html.spec.whatwg.org/commit-snapshots/578def68a9735a1e36610a6789245ddfc13d24e0/)
```

Which renders as:

[HTML](https://html.spec.whatwg.org/commit-snapshots/578def68a9735a1e36610a6789245ddfc13d24e0/)

Permitted WHATWG specification URLs must anchor to a specification defined in the `spec` subdomain (idea specifications are not allowed) and to a commit snapshot, and so must match this regular expression:

```regex
^https:\/\/[a-z]*\.spec\.whatwg\.org/commit-snapshots/[0-9a-f]{40}/$
```

Although not recommended by WHATWG, EIPs must anchor to a particular commit so that future readers can refer to the exact version of the living standard that existed at the time the EIP was finalized. This gives readers sufficient information to maintain compatibility, if they so choose, with the version referenced by the EIP and the current living standard.

### Internet Engineering Task Force (IETF)

Links to an IETF Request For Comment (RFC) specification may be included using normal markdown syntax, such as:

```markdown
[RFC 8446](https://www.rfc-editor.org/rfc/rfc8446)
```

Which renders as:

[RFC 8446](https://www.rfc-editor.org/rfc/rfc8446)

Permitted IETF specification URLs MUST anchor to a specification with an assigned RFC number (meaning cannot reference internet drafts), and so MUST match this regular expression:

```regex
^https:\/\/www.rfc-editor.org\/rfc\/.*$
```

### Bitcoin Improvement Proposal

Links to Bitcoin Improvement Proposals may be included using normal markdown syntax, such as:

```markdown
[BIP 38](https://github.com/bitcoin/bips/blob/3db736243cd01389a4dfd98738204df1856dc5b9/bip-0038.mediawiki)
```

Which renders to:

[BIP 38](https://github.com/bitcoin/bips/blob/3db736243cd01389a4dfd98738204df1856dc5b9/bip-0038.mediawiki)

Permitted Bitcoin Improvement Proposal URLs must anchor to a specific commit, and so must match this regular expression:

```regex
^(https://github.com/bitcoin/bips/blob/[0-9a-f]{40}/bip-[0-9]+\.mediawiki)$
```

### National Vulnerability Database (NVD)

Links to the Common Vulnerabilities and Exposures (CVE) system as published by the National Institute of Standards and Technology (NIST) may be included, provided they are qualified by the date of the most recent change, using the following syntax:

```markdown
[CVE-2023-29638 (2023-10-17T10:14:15)](https://nvd.nist.gov/vuln/detail/CVE-2023-29638)
```

Which renders to:

[CVE-2023-29638 (2023-10-17T10:14:15)](https://nvd.nist.gov/vuln/detail/CVE-2023-29638)

### Chain Agnostic Improvement Proposals (CAIPs)

Links to a Chain Agnostic Improvement Proposals (CAIPs) specification may be included using normal markdown syntax, such as:

```markdown
[CAIP 10](https://github.com/ChainAgnostic/CAIPs/blob/5dd3a2f541d399a82bb32590b52ca4340b09f08b/CAIPs/caip-10.md)
```

Which renders to:

[CAIP 10](https://github.com/ChainAgnostic/CAIPs/blob/5dd3a2f541d399a82bb32590b52ca4340b09f08b/CAIPs/caip-10.md)

Permitted Chain Agnostic URLs must anchor to a specific commit, and so must match this regular expression:

```regex
^(https://github.com/ChainAgnostic/CAIPs/blob/[0-9a-f]{40}/CAIPs/caip-[0-9]+\.md)$
```

### Ethereum Yellow Paper

Links to the Ethereum Yellow Paper may be included using normal markdown syntax, such as:

```markdown
[Ethereum Yellow Paper](https://github.com/ethereum/yellowpaper/blob/9c601d6a58c44928d4f2b837c0350cec9d9259ed/paper.pdf)
```

Which renders to:

[Ethereum Yellow Paper](https://github.com/ethereum/yellowpaper/blob/9c601d6a58c44928d4f2b837c0350cec9d9259ed/paper.pdf)

Permitted Yellow Paper URLs must anchor to a specific commit, and so must match this regular expression:

```regex
^(https://github\.com/ethereum/yellowpaper/blob/[0-9a-f]{40}/paper\.pdf)$
```

### Execution Client Specification Tests

Links to the Ethereum Execution Client Specification Tests may be included using normal markdown syntax, such as:

```markdown
[Ethereum Execution Client Specification Tests](https://github.com/ethereum/execution-spec-tests/blob/d5a3188f122912e137aa2e21ed2a1403e806e424/README.md)
```

Which renders to:

[Ethereum Execution Client Specification Tests](https://github.com/ethereum/execution-spec-tests/blob/d5a3188f122912e137aa2e21ed2a1403e806e424/README.md)

Permitted Execution Client Specification Tests URLs must anchor to a specific commit, and so must match this regular expression:

```regex
^(https://github.com/ethereum/execution-spec-tests/(blob|commit)/[0-9a-f]{40}/.*|https://github.com/ethereum/execution-spec-tests/tree/[0-9a-f]{40}/.*)$
```

### Digital Object Identifier System

Links qualified with a Digital Object Identifier (DOI) may be included using the following syntax:

````markdown
This is a sentence with a footnote.[^1]

[^1]:
    ```csl-json
    {
      "type": "article",
      "id": 1,
      "author": [
        {
          "family": "Jameson",
          "given": "Hudson"
        }
      ],
      "DOI": "00.0000/a00000-000-0000-y",
      "title": "An Interesting Article",
      "original-date": {
        "date-parts": [
          [2022, 12, 31]
        ]
      },
      "URL": "https://sly-hub.invalid/00.0000/a00000-000-0000-y",
      "custom": {
        "additional-urls": [
          "https://example.com/an-interesting-article.pdf"
        ]
      }
    }
    ```
````

Which renders to:

<!-- markdownlint-capture -->
<!-- markdownlint-disable code-block-style -->

This is a sentence with a footnote.[^1]

[^1]:
    ```csl-json
    {
      "type": "article",
      "id": 1,
      "author": [
        {
          "family": "Jameson",
          "given": "Hudson"
        }
      ],
      "DOI": "00.0000/a00000-000-0000-y",
      "title": "An Interesting Article",
      "original-date": {
        "date-parts": [
          [2022, 12, 31]
        ]
      },
      "URL": "https://sly-hub.invalid/00.0000/a00000-000-0000-y",
      "custom": {
        "additional-urls": [
          "https://example.com/an-interesting-article.pdf"
        ]
      }
    }
    ```

<!-- markdownlint-restore -->

See the [Citation Style Language Schema](https://resource.citationstyles.org/schema/v1.0/input/json/csl-data.json) for the supported fields. In addition to passing validation against that schema, references must include a DOI and at least one URL.

The top-level URL field must resolve to a copy of the referenced document which can be viewed at zero cost. Values under `additional-urls` must also resolve to a copy of the referenced document, but may charge a fee.

### Execution API Specification

Links to the Ethereum Execution API Specification may be included using normal markdown syntax, such as:

```markdown
[Ethereum Execution API Specification](https://github.com/ethereum/execution-apis/blob/dd00287101e368752ba264950585dde4b61cdc17/README.md)
```

Which renders to:

[Ethereum Execution API Specification](https://github.com/ethereum/execution-apis/blob/dd00287101e368752ba264950585dde4b61cdc17/README.md)

Permitted Execution API Specification URLs must anchor to a specific commit, and so must match this regular expression:

```regex
^(https://github.com/ethereum/execution-apis/(blob|commit)/[0-9a-f]{40}/.*|https://github.com/ethereum/execution-apis/tree/[0-9a-f]{40}/.*)$
```

## Linking to other EIPs

References to other EIPs should follow the format `EIP-N` where `N` is the EIP number you are referring to.  Each EIP that is referenced in an EIP **MUST** be accompanied by a relative markdown link the first time it is referenced, and **MAY** be accompanied by a link on subsequent references.  The link **MUST** always be done via relative paths so that the links work in this GitHub repository, forks of this repository, the main EIPs site, mirrors of the main EIP site, etc.  For example, you would link to this EIP as `./eip-1.md`.

## Auxiliary Files

Images, diagrams and auxiliary files should be included in a subdirectory of the `assets` folder for that EIP as follows: `assets/eip-N` (where **N** is to be replaced with the EIP number). When linking to an image in the EIP, use relative links such as `../assets/eip-1/image.png`.

## Transferring EIP Ownership

It occasionally becomes necessary to transfer ownership of EIPs to a new champion. In general, we'd like to retain the original author as a co-author of the transferred EIP, but that's really up to the original author. A good reason to transfer ownership is because the original author no longer has the time or interest in updating it or following through with the EIP process, or has fallen off the face of the 'net (i.e. is unreachable or isn't responding to email). A bad reason to transfer ownership is because you don't agree with the direction of the EIP. We try to build consensus around an EIP, but if that's not possible, you can always submit a competing EIP.

If you are interested in assuming ownership of an EIP, send a message asking to take over, addressed to both the original author and the EIP editor. If the original author doesn't respond to the email in a timely manner, the EIP editor will make a unilateral decision (it's not like such decisions can't be reversed :)).

## EIP Editors

The current EIP editors are

- Matt Garnett (@lightclient)
- Sam Wilson (@SamWilsn)
- Zainan Victor Zhou (@xinbenlv)
- Gajinder Singh (@g11tech)

Emeritus EIP editors are

- Alex Beregszaszi (@axic)
- Casey Detrio (@cdetrio)
- Gavin John (@Pandapip1)
- Greg Colvin (@gcolvin)
- Hudson Jameson (@Souptacular)
- Martin Becze (@wanderer)
- Micah Zoltu (@MicahZoltu)
- Nick Johnson (@arachnid)
- Nick Savers (@nicksavers)
- Vitalik Buterin (@vbuterin)

If you would like to become an EIP editor, please check [EIP-5069](./eip-5069.md).

## EIP Editor Responsibilities

For each new EIP that comes in, an editor does the following:

- Read the EIP to check if it is ready: sound and complete. The ideas must make technical sense, even if they don't seem likely to get to final status.
- The title should accurately describe the content.
- Check the EIP for language (spelling, grammar, sentence structure, etc.), markup (GitHub flavored Markdown), code style

If the EIP isn't ready, the editor will send it back to the author for revision, with specific instructions.

Once the EIP is ready for the repository, the EIP editor will:

- Assign an EIP number (generally incremental; editors can reassign if number sniping is suspected)
- Merge the corresponding [pull request](https://github.com/ethereum/EIPs/pulls)
- Send a message back to the EIP author with the next step.

Many EIPs are written and maintained by developers with write access to the Ethereum codebase. The EIP editors monitor EIP changes, and correct any structure, grammar, spelling, or markup mistakes we see.

The editors don't pass judgment on EIPs. We merely do the administrative & editorial part.

## Style Guide

### Titles

The `title` field in the preamble:

- Should not include the word "standard" or any variation thereof; and
- Should not include the EIP's number.

### Descriptions

The `description` field in the preamble:

- Should not include the word "standard" or any variation thereof; and
- Should not include the EIP's number.

### EIP numbers

When referring to an EIP with a `category` of `ERC`, it must be written in the hyphenated form `ERC-X` where `X` is that EIP's assigned number. When referring to EIPs with any other `category`, it must be written in the hyphenated form `EIP-X` where `X` is that EIP's assigned number.

### RFC 2119 and RFC 8174

EIPs are encouraged to follow [RFC 2119](https://www.ietf.org/rfc/rfc2119.html) and [RFC 8174](https://www.ietf.org/rfc/rfc8174.html) for terminology and to insert the following at the beginning of the Specification section:

> The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

## History

This document was derived heavily from [Bitcoin's BIP-0001](https://github.com/bitcoin/bips) written by Amir Taaki which in turn was derived from [Python's PEP-0001](https://peps.python.org/). In many places text was simply copied and modified. Although the PEP-0001 text was written by Barry Warsaw, Jeremy Hylton, and David Goodger, they are not responsible for its use in the Ethereum Improvement Process, and should not be bothered with technical questions specific to Ethereum or the EIP. Please direct all comments to the EIP editors.

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
