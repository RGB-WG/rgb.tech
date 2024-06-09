+++
title = "RGB v0.11 Beta 6 is out"
date = 2024-06-09
authors = ["Maxim Orlovsky"]
description = "A new beta release of Bitcoin and Lightning smart contract protocol"
+++

LNP/BP Standards Association is happy to announce the sixth beta release of RGB version 0.11.

The release marks a reference point for public audit of RGB v0.11. The APIs are now stable, and
feature-freeze has been reached: no further consensus-level or standard library breaking changes are
planned (unless they would become required as a result of newly discovered bugs/issues by the
community or auditors).

Our main focus with Beta 6 was:

- ensure with end-to-end tests that any execution routes and user stories doesn't contain bugs and
  preform correctly;
- refactor consensus and security-critical code in order to decrease its size and reduce potential
  attack surface as much as possible;
- make sure that RGB will be able to evolve in the future without introducing backward-incompatible
  changes.

All developers working with RGB are urged to update to the new version and use it to ensure
cross-application interoperability between assets, invoices, consignments and other RGB exchange
data.

Unless required, no further beta releases are planned and RGB v0.11 enters public testing and
pre-production preview cycle.


## What's new in Beta 6

Beta 6 contain one of the largest changelog: we have fixed, closed and merged more than 120 issues
and pull requests. More than 20 engineers, both from LNP/BP Standards Association member companies
and independent third-parties has contributed since last Beta 5 release by reviewing, discussing and
proposing PRs to the RGB codebase across ~50 repositories related to the project.

The complete list of all closed issues and pull requests can be found on the [public RGB roadmap
page][proj]; below we review the most important of them:

### Consensus-level changes

Beta 6 implements four proposals for consensus-level changes coming from the
[RGB change proposals][RCP]:

- [RCP-240227A]: Make timestamp part of genesis
- [RCP-240227B]: Make issuer identity part of genesis
- [RCP-240313B]: Move script and type libraries outside of schema, interfaces and operations
- [RCP-240326A]: Multiple metadata fields in contract operations

Other than that, we have:
- Moved information about asset tags, used in homomorphic encryption of the contract state, from
  standard library to contract genesis and adding them to the contract commitments; also 
  significantly simplifying making contract operations which involve these tags;
- Improved multichain support, such that RGB contracts can run on top of multiple layer 1 soultions.
  This gives contract users ability to do cheaper and faster transactions using Liquid network,
  and benefit from the new client-side validated layer 1 called Prime, scheduled for the release
  later;
- New virtual machine op-codes for introspecting smart contract state and simplified branching
  op-codes making scripts more streamlined;
- More RGB macro assembly instructions allowing to write complex verification code;
- Improved checking and handling of strings with restricted character set, like the ones used for
  RGB20 asset tickers and names;
- Refactored mechanism for combining different types of single-use seals (like tapret- and 
  opret-based) in inputs of a state transitions;
- Removed legacy schema inheritance feature in favor of new advanced interface inheritance, which
  is implemented outside of consensus code (see below).

Additionally, we have put a consensus changes required to make the following proposals 
fast-forwards, which means that they can be implemented in a future releases (like v0.12 etc) 
without consensus-breaking changes:

- [RCP-240313A]: SPV proofs for anchor transactions
- [RCP-240327A]: Computed state
- [RCP-240406A]: Fallback single-use-seal

We have also adjusted the complexity counter for reserved RGB op-code values, which ensures that
adding more op-codes to the virtual machine can be done in a backward-compatible manner.

Overall, with all those changes **the size of consensus code in RGB Core library has decreased from
7170 to 6726 lines** of Rust code (LORC), excluding comments and blank lines, which is 
**>6% reduction**.

### Interfaces and inheritance

Interface inheritance is the brand-new way of designing RGB smart contract interfaces which brings
much better modularity, audibility and upgradeability for the contracts.

Interfaces are defined at the level above consensus and specify how software or a user can interact
with a given contract. *Interface* defines a semantic for a business logic of a contract, and
contract developers write an **implementations** linking that business logic to the semantics. In
other words, interface explains to the user the meaning of a smart contract. One example is RGB20:
an interface (which becomes a family of interfaces in beta 6) which all contracts with a fungible
tokens can implement to be supported by any RGB wallet. The fact that interfaces are not a part of
the consensus level allows contract developers to gradually provide more and more advanced APIs
without a need to re-issue an assets or create new contract versions. On the other hand, users may
interact with smart contracts using newly released interfaces without the need of a wallet software
to be upgraded.

Previously RGB v0.11 beta 6 there were no interface inheritance, meaning that a contract can
implement a given interface only once; and must provide all of its functionality at once. Now, with
interface inheritance, this can become a gradual process. For instance, today RGB20 standard defines
what fungible assets can do. In a future a new features can be added to the standard (like reserves
or vesting), implemented as a new interfaces inheriting existing RGB20 interfaces and implemented
by existing assets which had that functionality in their business logic – and wallets will be able
to handle them in a correct way without any software upgrades.

As a part of the work on interfaces and inheritance, we have refactored the way errors can be
reported from smart contract validation scripts. Now, you can provide detailed error messages as
strings inside interface definitions and link them to raised errors via integer codes in
implementations.

At the same time interface inheritance brings much better modularity and audibility: you can compare
RGB20 standard 
[before the inheritance was introduced](https://github.com/LNP-BP/LNPBPs/blob/dc16fd091beaa74b171ce1d1fa04e902ef3a8012/lnpbp-0020.md)
and [now][RGB20].

We have re-implemented all previosly-defined interface standards (RGB-20, 21 and 25) as standard
sets ("families") of related interfaces and re-designed the APIs for working with them.

Finally, with beta 6 we added a number of interface and implementation integrity and validity
checks, which should simplify the work of developers and debugging.

The details of how the inheritance works can be found in
[RCP-240325A: Interface inheritance][RCP-240325A] proposal.

### Invoicing

We also have made few changes to the invoicing. In RGB, invoices are form of URIs (like URLs), and
thus can be opened with one-click in your favorite wallet.

With beta 6 we changed the format for displaying asset amount inside the invoice. RGB20 standard,
like many other asset standards in other systems (ERC20 etc) allow to specify a precision for asset
amounts. Invoices contain information about the amount with no precision, creating a lot of
confusion for users, as well as a attack surface. Starting from the current release we hide the
amounts behind custom latin letter encoding (modified Base32 encoding with no numbers used),
removing the problem. To read an invoice a user must use wallet or other software, which will parse
it for the user using correct precision from the contract information. To simplify working with such
amounts we provide a new `CoinAmount` type which embeds information about precision and is handy for
presenting asset values in the UI.

Finally, invoices has changed they way a pay-to-address requests are handled. Previously, an
explicit bitcoin address was used; now we standardized that with usual blind seal definitions using
the same encoding, chunking and supporting incorporation of custom seal close method information.

Last, but not least, we have optimized identities we use across RGB (like contracts, interfaces etc)
– it was switched from Baid58 to Baid64 encoding, which resulted in shorter fixed-length identifiers,
robust chunking, and solved few issues with its URI standard compatibility and chunking.

### Data containers

With client-side validation, all the contract state, new state transitions and history are sent
between peers, in a P2P way. This is quite different from the blockchain-bases systems and require
specific methods of storing and sharing the data: data containers.

There are several types of data containers in RGB. In v0.11 beta 6 we have put a lot of
optimizations into the way containers work, making them more compact and secure.

Previously, information for smart contract developers – interfaces, schemata and interface
implementations were always present as a separate files. With beta 6 we introduce **kits**: new data
container type which may store multiple interfaces, schemata, implementations and other data in one
single package. We have also made sure that repeated information – like data type or script
libraries which can be re-used by different interfaces and contracts – is shipped only once,
reducing the size of data containers for about a dozen of percents.

Another type of data containers are consignments, used to distribute smart contracts, assets and do
transfers. Consignments has also become smaller, since they also include all related interfaces and
implementations and thus do benefit from aggregation of data type and script libraries.

Previosly containers were packed with mechanism called *bindles*. With beta 6, we deprecate bindles,
which simplify the standard library API.

Special attention was paid to supplements: non-consensus meta information added to consignments,
and kits – like custom icons for assets etc. We have rewritten supplement API and made them really
extensible, such that new types of information and annotations can be added without requiring
updates to the standard library.

Data containers are binary files, but RGB supports a special ASCII-armored text representation using
Base85 encoding. We added few improvements to the ASCII armoring, like more informative headers,
which now may contain lists; verification of checksums and increased number of characters per line
to reduce file size and text length of the armored data.

Last, but not least: we are introducing ability for developers and contract issuers to provide their
identities and sign interfaces, contracts and other parts of the information, such that wallets can
use decentralized web of trust-like functionality to filter trusted and non-trusted sources and
protect users. The new identity system is flexible: it can support multiple standards, with PGP/GPG,
SSH are in pipeline. We also plan to use self-sovereign identities (SSI) developed by LNP/BP
Standrards Association in partnership with Cyphernet: a new single-use seal based web-of-trust
standard for compact and easy-to-use identities.

### Other standard library changes

There are more improvements in the standard library APIs, including completed deterministic contract
and operation creation; simplified API for working with contracts and consignments; many new
interface-related APIs and support for accessing and adding metadata and state extension-related
information to the contracts and operations.

### Application-level changes

For the first time, RGB v0.11 comes with embedded wallet support right out of the box. This was one
of the major pain points for the users and developers. Beta 6 extends and simplifies RGB wallet
APIs, as well as improves Electrum and Esplora support.

### Toolchain and dev tools

On top of standard and wallet libraries RGB v0.11 include a new high-level RGB runtime for software
integration and simple command-line tool `rgb` made with it. In beta 6 we put a lot of effort in
improving them further, providing new APIs and commands, as well as improving output presentation
and information details, including presenting non-asset smart contract state like digital rights,
attachments and arbitrary structured data. Also, we added commands for debugging and testing smart
contracts, like those converting data consignments and stash information into human-readable and
editable YAML format, also allowing to import it back with some changes; which can be useful for
penetration testing.

Client-side-validated data are now kept not as a single file, but as three independent files:
`stash.dat`, with critical non-recoverable consensus information; `index.dat` – indexes and
`state.dat` - cache of the contract state for faster access.

We are continuing our work on providing high-level programming language **Contractum** for RGB
interface and contract developers. In beta 6 we included a reverse compiler which translates any
binary interface, iterface implementation or a contract into Contractum representation. For
instance, all standard interface families, like [RGB20] etc, were generated with it.

One of the most security-important things coming with beta 6 is the integration end-to-end tests
developed and contributed by [Bitfinex] RGB team. They cover about a hunderd of different user
stories, involving assets issuance and complex multi-asset multi-wallet transfers, involving
different forms of single use seals and their combination. This tests were the major tool of
debugging the new release and with them passing we can now proceed to the next stage: audit and
penetration testing.


## What's next

With Beta 6 release RGB development enters a new phase of preparation for a public preview of
the RGB v0.11. Beta 6 is recommended to be used by all parties integrating RGB, and after gathering
their feedback (and fixing all newly discovered issues) we will release the public preview version.
In parallel, with support from [Bitfinex], we are starting working on audit and petentration testing
of RGB, ensuring the system security and safety before the final release.

We continue an ongoing effort of documenting RGB, and with beta 6 stabilizing consensus and standard
APIs we plan to review, update and extend all existing documentation and prepare it for a reliease
alongside the v0.11.

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
- Scripting, with new state introspection codes and RGB assembly compiler;
- Initial support for Contractum language;
- Ability to inherit interfaces (multiple inheritance!);
- Support of Electrum RPC additionally to Esplora REST, present before;
- Commitments to issuer and developer identities embedded into contracts, interfaces, libraries;
- More compact consignments and better ASCII armoring.



## Contributing

RGB is a free software and opensourc effort, built by community. The development is supervised by
a non-profit [LNP/BP Standards Association][LNPBP], which uses open approach: anybody from the
community can propose a request for RGB change ([RCP]), ask a question in [discussions], open an
issue in GitHub, propose a pull request or review pull requests of others. The association also
maintains public [roadmap][proj] summarizing the current development progress and listing all issues
and pull requests across multiple repositories and github organizations related to the RGB.

Specifically, starting from the beta 6 release, we are also looking for qualified auditors and
penetration testers who can contribute into making RGB more secure and robust. We plan a grant
program to facilitate this process; with any enquiries please contact us via <info@lnp-bp.org>.


## Acknowledgements

We are grateful to [Fulgur Ventures], providing another year of support for our efforts in
developing RGB. This year they were joined by [Bitlight Labs], who had become a new full member of
the Association.

We have received a number of important contributions and bugfixes coming from commercial
companies, such as [Pandora Prime], [Bitfinex] and [DIBA]. We are also grateful for individual
contributors, who do their small -- but still highly valuable and welcomed input in making RGB
better.



[RCP]: https://github.com/RGB-WG/RFC/issues
[RCP-240227A]: https://github.com/RGB-WG/RFC/issues/1
[RCP-240227B]: https://github.com/RGB-WG/RFC/issues/2
[RCP-240313A]: https://github.com/RGB-WG/RFC/issues/3
[RCP-240313B]: https://github.com/RGB-WG/RFC/issues/4
[RCP-240326A]: https://github.com/RGB-WG/RFC/issues/5
[RCP-240327A]: https://github.com/RGB-WG/RFC/issues/6
[RCP-240406A]: https://github.com/RGB-WG/RFC/issues/7
[RCP-240325A]: https://github.com/RGB-WG/RFC/issues/8
[discussions]: https://github.com/orgs/RGB-WG/discussions
[proj]: https://github.com/orgs/RGB-WG/projects/17/views/1
[RGB20]: https://github.com/RGB-WG/rgb-interfaces/blob/master/interfaces/RGB20.con
[LNPBP]: https://www.lnp-bp.org
[Fulgur Ventures]: https://fulgur.ventures
[Bitlight Labs]: https://bitlightlabs.com
[Pandora Prime]: https://pandoraprime.ch
[Bitfinex]: https://www.bitfinex.com/about
[DIBA]: http://diba.io

