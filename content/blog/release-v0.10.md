+++
title = "RGB release v0.10"
date = 2023-04-10
+++

TL;DR
-----

LNP/BP Standards Association <https://www.lnp-bp.org>, supported by Hojo
Foundation, Fulgur Ventures, Pandora Prime, Bitfinex and DIBA, is happy to
announce the release of RGB v0.10 - the next significant milestone in RGB
protocol <https://rgb.tech> which brings full support to smart contracts to
Bitcoin and Lightning. This is a result of a great cross-industry long-term
collaboration between these bitcoin companies and more than four years of
extensive development work.

RGB v0.10 can be downloaded and installed as described on <https://rgb.tech>
website, which also contains a number of user and developer guidelines.
RGB source code can be found on <https://github.com/RGB-WG>


Background of RGB
-----------------

Some of you might remember an announcement of RGB protocol idea back in
2018 [1]: it was an idea of “colored coins” over the lightning by Giacomo
Zucco, based on new concepts developed by Peter Todd - client-side-validation
and single-use-seals.

In 2019 me (Maxim Orlovsky) and Giacomo Zucco has formed an LNP/BP Standard
Association, aiming bring RGB from the concept to production. The initiative
was supported by Bitfinex and Fulgur Ventures. My goal with RGB was not just
enable assets on Lightning, but of a much larger scope: to build a
programmability layer for Bitcoin and Lightning, which may unlock much more
than just tokens: DAOs, decentralized identities and other cases lacked in
bitcoin itself. This has taken much longer than was expected, and me &
LNP/BP Standards Association had went through very complex times on this
road, relying on self-financing for more than a year…

Nevertheless, in 2021 we were able to present both RGB powered with a
Turing-complete virtual machine (AluVM) [2] and RGB operations [3] on a
Lightning Network using a LNP Node - a complete rust re-implementation of
Lightning protocol made by me at the Association [4]. Those who are
interested in the history of RGB development and our past releases may refer
to it graphical representation [5] - or navigate through three years of
videos and demos on our YouTube channel
<https://www.youtube.com/@LNPBP/videos>.

Despite 4 years of active development, weekly community calls, talks on
all mainstream bitcoin-only evens and conferences, the awareness about RGB
in the bitcoin community is still very small - and some bitcoin media put
as a requirement for us to submit information about RGB to bitcoin-dev mail
list, so they can see a new technology has been developed - so here we are.


RGB v0.10
---------

Today we’d like to announce a next main milestone: **release of RGB v0.10**,
which includes consensus layer, standard library (used by wallets/exchange
in integration) and command-line tool.

Release v0.10 is a major milestone which brings RGB further to be a production-
ready system. It introduces the last breaking consensus changes, aiming at
keeping future RGB versions fully backward-compatible. It also unlocks last
features which were required for implementing fully-functional smart contracts,
which may be arbitrary customized by contract developers.

RGB v0.10 can be downloaded and installed as described on [https://rgb.tech]
(https://rgb.tech) website, which also contains a number of user and
developer guidelines. RGB source code can be found on https://github.com/RGB-WG

This release brings support of the following features to the RGB:

- ### Global state in RGB contracts
  Now each RGB contract has a global state accessible by a virtual machine
  and clients (wallets etc).

- ### Contract interfaces
  Interfaces, introduced in this version, represent a standard way of
  communicating a diverse range of smart contracts through a well-defined APIs.
  Interfaces can be compared to contract ABIs and ERCs in Ethereum world,
  however, unlike in Ethereum, they neither require obligatory standardization
  (as ERCs) nor separate distribution, being always packed together with
  contracts. By using interfaces, wallets and other software can provide a
  semantic-aware UI for the users for working with the contracts - and contract
  developers may add more interfaces to their existing contracts over time 
  without a need to update the immutable contract itself.

- ### Strict type system
  Strict types is a new functional data type system with provable
  properties used for RGB contract state representation and introspection.
  It allows compile-time guarantees on the size of any data, simplifying RGB
  operations on low-end and limited-memory devices like hardware wallets.
  The whole RGB consensus layer is now compiled into strict types, which
  allows formal proves of the binary compatibility between releases (a
  feature which may be very useful for bitcoin consensus if it existed back
  in the days of Satoshi). You can learn more about strict types on
  <https://www.strict-types.org>

- ### Contracts in Rust
  Writing and compiling RGB smart contracts in Rust. Thanks to strict types,
  it is also possible to compile rust data types right into RGB contracts.

- ### State introspection
  Contracts can introspect own state in validation code used by the virtual
  machine, which unlocks a way for writing complex forms of contracts working
  with bitcoin transactions, DLCs and other complex data.

- ### URL-based invoice format
  Previously RGB was used Bech32m-encoded invoices, which were very long,
  not human-readable and can’t be automatically opened with the most of the
  software. The new format is much shorter, easier to verify by a user and
  open automatically as a link with a preconfigured software.

- ### WASM support
  RGB standard library can run without I/O and file system access,
  i.e. can operate inside a web page or browser plugin.

- ### Tapret descriptors and custom derivation
  RGB uses taproot-based OP_RETURN commitments (shortly, *tapret*), which
  require support at the descriptor level such that wallets can see transactions
  with tweaked outputs as belonging to the wallet descriptor. New version also
  introduces custom derication indexes with prevents non-RGB wallets from
  accidentally spending outputs with RGB assets (destroying assets).

- ### Simplified dependencies
  RGB consensus layer ships with fewer dependencies, improving stability of API.
  We have abandoned dependency on custom bulletproofs implementation from
  Grin projects. We also do not use rust-bitcoin and rust-miniscript due to
  overall API instability and recently discovered bugs; since RGB uses very
  small subset of bitcoin functionality it is now implemented as a part of the
  library with no assumptions about bitcoin consensus layer (like those which
  halted rust-bitcoin powered software when Burak's hack had happened last 
  year).

- ### Simplified integration
  Many operations which previously required multiple API calls, as well as
  cross-language encoding of complex data structures are now work with a
  single API call. RGB contract state is represented as a JSON object and
  can be serialized across different languages without a hassle.

- ### Simplified UX
  Previously, to use RGB, a wallet or user had to run RGB Node, interface it
  through RPC (or cli tool) - and use a number of other libs and
  command-line tools to perform most of the operations on PSBTs etc. With
  the new release this complex stack was replaced with single library API
  and a command-line tool `rgb`, which operates like a swiss knife for RGB
  user (and it can be compared to the way `git` works). RGB Node still may
  be run by users on their home servers, but is not anymore required to use RGB.

### Migration notes

There is no migration from contracts issued on RGB v0.9 to the future versions.
All assets have to be re-issued; asset holders can contact asset issuers to
provide them a newly re-issued contracts and assets matching the assets from
v0.9.


Roadmap after v0.10
-------------------

With this release future development of the core RGB technology (at its
consensus layer) becomes gradually ossified: in case of
client-side-validated systems upgrades are more complex to coordinate than
in blockchain layer 1. Also, normal understanding of soft-forks and
hard-forks to not apply to upgrades in layer 2 and 3. So we have found a way
for backwards-compatible upgrades, we call “fast-forwards”, where users keep
their assets issued under older versions always operatable and acceptable by
users of any other future version, however users of a newer version will be
restricted in transferring their assets only to the users of the same or
more recent version (but they can always ask recepients to upgrade their
software). We have a number of features planned for the future fast-forwards:

- full support of bitcoin layer 1 and channel state introspection
- inter-contract interaction
- support for bulletproofs++
- zero-knowledge-based optimizations of client-side-validated history size

We're also working on the design of a layer 1 which will be perfect for the
client-side-validated applications (“how to design a blockchain today if we
knew about client-side-validation/single-use-seals”). This should be very
compact (order of one signature per block) ultra-scalable (theoretically
unlimited no of tx in block) chain which may run systems like RGB - with
Bitcoin UTXO set migrated into RGB operating on both bitcoin blockchain and
this new chain (we code-name it “sigchain”). However, these are quite early
developments with a number of unsolved tradeoffs and challenges; if there is
an interest on this topic here we may set up a different discussion thread
on the matter.


Software and integrations
-------------------------

Companies, which are the part of the LNP/BP Standards Association, as well
as other independent vendors are already working on upgrading their software
to v0.10. This includes (but not limited to):

- MyCitadel wallet (iOS, Desktop) - mycitadel.io
- Iris wallet (Android) from RGB team inside Bitfinex
- DIBA (Web browser plugin) from BitMask
- LNP Node with RGB support - lightning node from LNP/BP Standards Association
- RGBEx.io - website listing RGB assets and contracts,
  using pseudonymous credentials (public key + signatures)
- RGB Tools library - SDK based on BitcoinDevKit for RGB wallet integration
- LightningDevKit with RGB support - again by RGB team in Bitfinex
- Core Lightning RGB plugin

LNP/BP Standards Association will focus its further development activity
around these three areas:

- Completing RGB documentation, specification and helping public audits of the
  technology. Right now we have nearly-complete RGB whitepaper [6] and
  codified standards [7]
- Lightning network support for complex RGB smart contracts - a thing we name
  #BiFi (Bitcoin finance) [8]. It includes further development of Storm -
  a decentralized data storage & messaging network on top of Lightning, we have
  presented last year.
- RGB toolchain, which includes a new high-level functional smart contracting
  language called Contractum (<https://www.contractum.org>) and other tools
  which will simplify life of RGB devs.


Sources of information
----------------------

We have already witnesses some actors distributing misinformation about
non-existing products released with RGB even before official releases of
some of RGB components. Thus, we advise all media people to always check
with the official sources before publishing information about RGB protocol.
All releases and major things are announced on:
* Official website of the protocol <https://rgb.tech>,
  community <https://rgbfaq.com> and LNP/BP Standards Association
  <https://lnp-bp.org>.
* Twitter of the Association: <https://twitter.com/lnp_bp>
* Community telegram channel: <https://t.me/rgbtelegram>

Other websites are not in control of the RGB protocol developers and not
open-source and should not be considered as a trusted sources of information.


-------------------------
Maxim Orlovsky
Mail: orlovsky [at] lnp-bp.org
Github: https://github.com/dr-orlovsky
Twitter: https://twitter.com/dr_orlovsky
PGP: EAE7 30CE C0C6 6376 3F02 8A58 6009 4BAF 18A2 6EC9


[1]: https://www.youtube.com/watch?v=xHWxtmgQP94
[2]: https://www.youtube.com/watch?v=Mma0oyiVbSE
[3]: https://www.youtube.com/watch?v=6Svmh0OQVf4
[4]: https://github.com/LNP-WG/lnp-node
[5]: https://twitter.com/dr_orlovsky/status/1640833926456307714/photo/1
[6]: https://blackpaper.rgb.tech
[7]: https://standards.lnp-bp.org
[8]: https://www.youtube.com/watch?v=DtkTE6m0zio_
