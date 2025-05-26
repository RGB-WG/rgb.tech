+++
title = "RGB v0.12 release candidate 1"
date = 2025-05-20
authors = ["Maxim Orlovsky", "Olga Ukolova"]
description = "RGB 0.12 reaches release candidate stage, bringing new era to the Bitcoin and Lightning smart contracts"
+++

LNP/BP Standards Association announces that RGB version 0.12 has reached the release candidate stage.

All RGB protocol developers and integrators are recommended to start the integration with v0.12 RC1.
Due to the consensus-level changes, contracts issued before this release will not be compatible with
the new RGB version.

TL;DR
-----

RGB just got one step from production with the first release candidate of its most tested and
robust version (0.12).
Over the last half of the year, the 0.12 version has brought zk-STARK readiness to the RGB
(making it "zk-STARKy"), a huge performance boost, and a lot of simplification and security
improvements. The release candidate is for all developers working with RGB, so they can integrate
it into their software, having guarantees of consensus and low-level API freeze.

----------------------------------------------------------------------------------------------------

What's new
----------

The version 0.12 of RGB represents a result of 7 months of protocol re-design,
aimed at the following:
1. Make protocol ready for zk-STARK compression;
2. Simplify the protocol as much as possible,
   reducing any potential attack surface and helping protocol adoption;
3. Ensure security of the protocol with thorough unit and integration test coverage,
   including tests for potential attacks;
4. Make protocol fully production-ready by freezing consensus level changes and ensuring robustness
   and performance.

### 1. zk-STARKky RGB

The new RGB architecture for the first time introduces zk-AluVM, which is a compact and performant
Turing-complete zk-VM made for client-side validation.
While being universal, it offers a non-von-Neuman architecture with an extra-reduced instruction set,
made of just 40 instructions and provides read-once memory compatible with single-use seals.

The contract state was also transformed. Previously, RGB contracts had three variants of state data:
fungible (with Pedersen commitments and Bulletproofs), non-fungible structured state and binary
attachments. To make the contract state zk-compressible, version 12 unified state into a single
type, made of finite field elements.

The whole consensus validation was reduced to just a few hundred lines of code, which can be
represented as an arithmetic circuit for a zk prover, allowing recursive history compression.

All these changes not just make RGB ready for zk-STARK provers and verifiers, but also allow
use of RGB with the future client-side validation layer 1 codenamed [Prime].

### 2. Protocol simplification

RGB so far has been notoriously infamous for the protocol complexity.
Version 0.12 should shift that impression, since most of the changes in it
were an actual simplification of the protocol.

Overall, we have achieved x4 reduction in the size of the consensus code and
x2 reduction in standard libraries, as well as a significant reduction in the number of data types,
and removal of ~30% of generic type parameters in the APIs.

![Consensus codebase](/blog/v0-12-consensus.png)
![Standard libraries codebase](/blog/v0-12-std.png)

Such simplification provides several gains:
- much reduced attack surface;
- better auditability;
- better performance;
- better developer experience;
- which opens a way for building simpler UX with improved user experience.

Below, we cover some of the technical details of the main simplifications
which has happened in v0.12.

#### 2.1. Seal unification

RGB uses single-use seals, a cryptographic primitive, to prevent double-spending by "anchoring"
into a UTXO-based blockchain or a sidechain.

Previously, there were two different types of single-use seals, which can be used by an RGB contract:
so-called _tapret_ and _opret_-based. _Tapret_ seals can't be seen inside a bitcoin transaction,
but require a taproot wallet, which still is not fully supported by all exchanges,
hardware wallets or Lightning nodes. _Opret_ is an answer to that; it utilizes old `OP_RETURN`s
to put a 32-byte commitment into a bitcoin transaction;
which has many drawbacks (higher transaction price), exposure to chain analysis, etc.,
but helps when a wallet or a user can't use taproot.

However, the presence of two distinct seal types, where a user chooses one of the variants,
had created tremendous complexity, both in code, APIs and in user experience,
and resulted in a large attack surface.

In v0.12 we found a beautiful way how to unify two distinct seal types into a single-one,
yet still supporting `OP_RETURN` fallback option.

#### 2.2. Removal of Pedersen commitments and Bulletproofs

Since the inception of RGB protocol, it was using confidential assets by Blockstream for providing
contracts with a fungible state. While the idea of confidential assets has been great,
it was lacking an efficient implementation, postponing use of RGB in production.

The problem was that the original Blockstream implementation of confidential assets for Liquid
(and the Elements project) used older range proofs instead of bulletproofs,
which are much larger and longer to validate.
On the other hand, Bulletproofs were never implemented for the Bitcoin SECP256K1 curve,
while the RGB community and devs have been waiting for this implementation for more than 5 years.

With the introduction of zkVM and zk-STARK compatibility, Bulletproofs and Pedersen commitments
have become unneeded, since zk-STARK can provide much better performance 
and offers a range of ready-to-use implementations.

#### 2.3. State unification

With the removal of Pedersen commitments and Bulletproofs, and the introduction of zkVM,
we were able to simplify and unify the state of RGB contracts.
Previously, an RGB contract may have three forms of a state:
fungible, structured (non-fungible) and blob attachments.
In v0.12, this state is unified under a single state type, which, again,
reduces the codebase and the attack surface.

#### 2.4. No more schemata

zk-AluVM has changed a lot: with zk-STARKs, we need to be able to represent everything in the
consensus in an arithmetized form.
Thus, we got rid of the old way of writing RGB contracts with schemata and unlocked the real
potential for smart contract programming with Turing-complete zk-AluVM assembly
and higher-level languages on top, like Contractum.

From the point of view of contract issuers, now, instead of importing schemata, interfaces and
other components, they will be using so-called issuers, provided by the contract developers.
The users will be also less confused with an obscure terminology.

#### 2.5. Removed interfaces and implementations

The way how interfaces work was also fully refactored, mostly due to zkVM.
Most importantly, it has become much simpler.

Previously, software devs integrating RGB were lost in multiple types related to interfaces:
interfaces, standards, implementations, schemata, etc.
Many of them were also made with generic type parameters, which made it much harder to integrate
and fully support their functionality.

Now, with v0.12, the only thing a developer using RGB should know about is so-called contract _API_:
a simple non-generic object explaining to a wallet how to operate with a contract.

Interfaces are still present, but not in the standard library:
they live solely now in the world of Contractum language and are used only by contract developers,
and the Contractum compiler will be taking care of compiling them into the contract APIs.

#### 2.6. Single-blockchain protocols

Since v0.11, RGB supported multiple blockchains. In v0.12 we made it simpler: instead of allowing
a contract to operate on multiple blockchains, each blockchain will have its own version of the
contract, which significantly reduces possible attack surface and API complexity.

#### 2.7. No more blank state transitions

One of the concepts that were hard to grasp in RGB was the concept of blank state transitions.
In v0.12, because of new and better way to build contract API, they are gone.

### 3. Payment improvements

#### 3.1. Invoicing

The invoices have become much richer: 
they may specify any forms of a fungible- and non-fungible state 
and allow providing more accessory information in a better structured way.

#### 3.2. Multiple-asset contracts

Contracts now may expose multiple tokens, which may interact,
and these tokens can be accessed independently or jointly via both invoices and APIs.

#### 3.3. Payment scripts

Version v0.12 for the first time introduces a concept of _payment scripts_,
which is a powerful tool helping to construct and execute complex payments,
which may include multiple beneficiaries, multiple contracts and even multiple consequent
transactions or a transaction graph.
The functionality helps in:
- multisig wallets;
- coinjoins and payjoins;
- transaction batching at aggregation in wallets and exchanges;
- lighting channel operations;
- future Ark applications.

#### 3.4. Re-org support

As any blockchain-based system, RGB depends on what happens there, and one of the main
issues with blockchain deep reorgs, which may lead to the loss of funds by the users.

The problem was never properly addressed in RGB,
until in 2024 we discovered a type of attack which can leverage even shallow reorgs.

For the version v0.12 we have developed and implemented a mathematical model,
formally proving how the system can be protected from such attacks,
even if multiple and deep re-orgs happen or got combined.

### 4. Performance

Performance was one of the pain points of the past RGB release.
In version 0.12 we did a huge work to bring it up to the production requirements.

#### 4.1. Consignment streams

Transfer of client-side data during RGB payment is made in the form of a binary data sent
from a payer to a beneficiary, possibly via a relay.
These binary data packs are named _consignments_.

Before, consignments were distributed in the form of files,
and, before being validated and accepted, have to be fully read into memory by a receiver.
Since consignments may span multiple megabytes, this was putting high resource requirements
and made it nearly impossible to use hardware wallets for verification.

In v0.12 consignments have become streams, which are never put into the memory and are
validated on the go, as they are received,
not ever taking more than a few hundred bytes of memory (!!!).
This is much more suitable for mobile wallets, servers, and paves the way for the hardware wallet
applications.

#### 4.2. Consignment verification

It is not that the consignments are no more read into the memory;
during their validation, the RGB software doesn't make requests to a blockchain indexer.
Before, the consignment validation was slowed manifold due to the requests;
now, the blockchain index synchronization is separated from the consignment validation,
which became order of magnitude faster.

#### 4.3. Contract data are no more kept in memory

RGB runtime was also used to load and keep all the contracts and their data in the memory,
including the whole of the contract history.
This was boosting memory usage outside acceptable bounds, especially in server and mobile apps.

With v0.12, only used contracts are loaded, but even for them, only the index data and state
are kept in the memory and can be offloaded any time, contract-by-contract.
This reduced memory requirements several orders of magnitude.

#### 4.4. Persistence performance boost with NoSQL

Last, but not least, we have integrated a dedicated append-only NoSQL embedded database to
keep the contract history.
Since the database was designed specifically for the client-side validation needs,
where we do not need some operations like deletion, and many data have a known fixed size,
its introduction has boosted performance, not just compared to v0.11,
but also compared to classical embedded NoSQL databases like Rust Sled and ReDB.

### 5. Test coverage

In version 0.10, RGB for the first time got a full integration test suite.
However, we understood that it is not enough: integration tests,
demonstrating just the typical happy-path or shallow attack scenarios are insufficient
to prove the security of the system.

Version 0.12 is the first version achieving full consensus test coverage not just with
integration, but also with unit tests, which also check known attack scenarios.
The unit test coverage has increased several folds 
and exceeds 66.7% for all RGB abstraction levels, reaching >75% in the consensus libraries.

![Test coverage](/blog/v0-12-tests.png)

----------------------------------------------------------------------------------------------------

Using the release
-----------------

The source code of the release is available on [GitHub](https://github.com/RGB-WG/rgb/tree/v0.12.0-rc.1.1).

Those power users who'd like to play with the command line
may install it either from the sourcecode or using Rust Cargo command

```bash
cargo install rgb-wallet --version 0.12.0-rc.1.1
```

Demo & examples can be found here:
- https://github.com/RGB-WG/rgb/tree/v0.12.0-rc.1.1/examples
- https://github.com/RGB-WG/rgb-sandbox/blob/v0.12/demo.sh
- https://github.com/pandora-prime/rgb-issuers/

To start integrating RGB, please feel free to contract LNP/BP Standards Association
via e-mail info at lnp-bp dot org.

----------------------------------------------------------------------------------------------------

What's next
------------

1. We plan to update the RGB documentation and the website in order to reflect all the changes.
   This will be happening over the course of the next few weeks.

2. We will be supporting developers integrating RGB v0.12 into their products.
   Once we see the ecosystem adoption, v0.12 release candidate will become a full release.

3. We expect the first products and assets using this version to be released during June.
   While it is up to each company to decide when to launch their assets and apps
   into the mainnet, we hope that will happen during the Summer 2025.

4. Once v0.12 gets to the mainnet and production, a new non-profit, named __RGB Consortium__
   will be created by the companies using RGB in their products.
   This organization will ensure the protocol stability and will take care of the codebase maintenance.

----------------------------------------------------------------------------------------------------

Acknowledgments
---------------

The version was designed and implemented by Dr Maxim Orlovsky, financed by [Pandora Prime Inc.].
[Bitlight Labs] has contributed to the new version development by allocating a team of rust
developers, who also had completed the integral test coverage, ensuring the protocol security.
The initial three months of development were also financed by [Fulgur Ventures].

[Pandora Prime Inc.]:  https://pandoraprime.ch/
[Fulgur Ventures]: https://fulgur.ventures/
[Bitlight Labs]: https://bitlightlabs.com/
[Prime]: https://github.com/LNP-BP/layer1
