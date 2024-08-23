+++
title = "RGB v0.11 Beta 7 is out"
date = 2024-08-20
authors = ["Maxim Orlovsky"]
description = "A new beta release of Bitcoin and Lightning smart contract protocol"
+++

LNP/BP Standards Association is happy to announce the seventh beta release of RGB version 0.11.


## What's new in Beta 7

### Consensus-level changes

Beta 7 completes an important feature which will be required by any DeFi smart contract:
introspection of the contract state from inside AluVM scripts. While before it was possible to
access  state from the current operation only, in this release we have implemented a new engine
for dynamically computing and updating global contract state during the validation itself, such
that scripts can also access global state and outputs known to the contract.

This feature comes hand-in-hand with an improved ordering of the global state data. We introduce a
new `nonce` field in RGB operations, which allows to enforce the same deterministic ordering for
mempool, lightning channel and on-chain transactions. You can read more about this feature in
[RCP-240731A] proposal.


### Improved persistence

Previously, the state for each contract have to be fully stored in the memory. Beta 7 introduces a
new contract state engine, which works with the consensus level, but allows to keep the state in
some external storage, like file, database etc., not fully loading it into the memory. This allows
to ensure that validation can run efficiently in all environment, from enterprise setups to 
restricted embedded systems, like hardware wallets.

Beta 7 adds support for automatic detection of contract and stash data changes and automated saving
of the modified data, as well as recovery from the I/O failures, ensuring that the persisted state
is always consistent. This should work well with database environments and their transaction-commit-
rollback workflows.

### Support for re-orgs and RBFs

For the first time we added ability to the RGB wallet library to support both RBF transactions and
properly react on blockchain re-orgs, whenever they happen. This was achieved due to the complete
refactoring of how the RGB wallet operates. Previously, wallet was tracking only bitcoin 
transactions belonging to the wallet descriptor; however in RGB validity of operation also depends
on a second bitcoin transaction, called "witness transactions", which contain the commitment for the
single-use seals -- the mechanism by which RGB prevents double-spending of the client-side state.
In order to be valid, RGB state must belong to an operation output for which all previous history
of the operation graph is commited by the witness transactions included into the blockchain and
lightning state channels. As you see, a re-org, RBF transaction change or Lightining channel update
changes that validity, and the new wallet engine allows us to efficiently track those changes in
automatic fashion without creating excessive blockchain indexer requests.

### Support for hot wallets

Since RGB uses a novel type of bitcoin transaction commitments, **taprets** (OP_RETURN commitments
residing inside taproot tree), RGB taproot wallets are not supported by rust-bitcoin, miniscript and
BDK libraries. Thus, starting from 2022, [LNP/BP Association][LNPBP] was working on new and modern
rust implementation of bitcoin protocol named **BP libraries**. This release for the first time
ships with new version of BP supporting creation of hot wallets, which removes the last obstacle
in working with RGB in taproot.

### Improved Lightning functionality

Support of the Lightning network was significantly improved in this release; including ability to
efficiently handle RGB assets after a non-cooperative channel closings and ability to handle
state channel updates which doesn't modify RGB state -- a feature important when you perform a
bitcoin payment inside an RGB channel, or when the channel operates multiple RGB assets.

### Invoice changes

RGB20 fungible assets, alike many other token standards, may have a different precision, defining 
the asset divisibility. For instance, Bitcoin has precision of 8 digits (1 sat = 0.00000001 BTC),
USDT -- 6 digits etc. RGB invoices specify the amount of the assets to transfer as an integer, i.e.
as an amount of the smallest asset units. This may confuse users, since the transfer of 100 USDTs
will specify 100000000 as the amount. In order to avoid the confusion, starting the beta 7 the
specific asset amount in the RGB invoices is encoded in a non-human readable form.

### Other wallet improvements

Beta 7 adds support for Mempool extensions to the Esplora REST APIs, allowing using it as a backend
indexer for RGB wallets.

Many developers experience issues with Bitcoin Testnet, due to its spam pollution. A new proposal
of testnet re-load, named Testnet4, gains more and more traction. The current release adds support
of Testnet4 network to all RGB and wallet libraries.

In this version we also added ability to automatically resolve collisions between different 
commitment methods which may happen due to user mistakes. Now, if your taproot-based wallet will
receive an RGB payment with non-tapret single-use seal (which may happen also due to an adversarial
payee) the wallet will be able to see and use those funds with no manual recovery or adjustments.


## What's next

Beta 7 release continues a phase of preparation for a public preview of the RGB v0.11. All parties
are strongly recommended switching to this latest beta, which contains a number of critical bugfixes,
and provide us feedback their feedback.

Beta 7 was a result of internal audit and testing effort, we do together with [Bitfinex Labs], which
will continue with external audit in the following week. In this way we plan to ensure the system
security and safety before the final release.

You can track us on our journey towards v0.11 release with this [GitHub dashboard][proj].


## About RGB v0.11

RGB v0.11 is an evolution of the protocol coming with a lot of bug fixes and improvements, further
enhancing RGB smart contracting capabilities. It will be the first version which will be audited by
independent auditors, after which it can be considered by third-party issuers for the use in
production.

The main features shipping in v0.11 are:

- Wallet integration right into RGB runtime and command-line tool;
- Basic support for Liquid sidechain, with ability to add more alternative scalability layers in the
  future;
- Scripting, with new contract state introspection codes and RGB assembly compiler;
- Initial support for Contractum language;
- Ability to inherit interfaces (multiple inheritance!);
- Support of Electrum RPC and Mempool REST additionally to Esplora REST, present before;
- Commitments to issuer and developer identities embedded into contracts, interfaces, libraries;
- More compact consignments and better ASCII armoring.


## Contributing

RGB is a free software and opensourc effort, built by community. The development is supervised by
a non-profit [LNP/BP Standards Association][LNPBP], which uses open approach: anybody from the
community can propose a request for RGB change ([RCP]), ask a question in [discussions], open an
issue in GitHub, propose a pull request or review pull requests of others. The association also
maintains public [roadmap][proj] summarizing the current development progress and listing all issues
and pull requests across multiple repositories and GitHub organizations related to the RGB.

Specifically, we are also looking for qualified auditors and penetration testers who can contribute
into making RGB more secure and robust. We plan a grant program to facilitate this process; with any
enquiries please contact us via <info@lnp-bp.org>.


## Acknowledgements

We are grateful to [Fulgur Ventures], continuing their multi-year support for our efforts in
developing RGB, as well as other Association members, including [Bitlight Labs], who had become a
major contributor to RGB development. We are grateful to all commercial companies, building on RGB,
and providing their support, contributions and feedback, including [Bitfinex], [Pandora Prime], and
[DIBA]. We are also grateful for individual contributors, who do their small -- but still highly
valuable and welcomed input in making RGB better.


[RCP-240731A]: https://github.com/RGB-WG/RFC/issues/10
[discussions]: https://github.com/orgs/RGB-WG/discussions
[proj]: https://github.com/orgs/RGB-WG/projects/17/views/1
[RGB20]: https://github.com/RGB-WG/rgb-interfaces/blob/master/interfaces/RGB20.con
[LNPBP]: https://www.lnp-bp.org
[Fulgur Ventures]: https://fulgur.ventures
[Bitlight Labs]: https://bitlightlabs.com
[Pandora Prime]: https://pandoraprime.ch
[Bitfinex]: https://www.bitfinex.com/about
[DIBA]: http://diba.io
