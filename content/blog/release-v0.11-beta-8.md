+++
title = "RGB v0.11 Beta 8 is out"
date = 2024-09-05
authors = ["Maxim Orlovsky"]
description = "A new beta release of Bitcoin and Lightning smart contract protocol"
+++

LNP/BP Standards Association is happy to announce the eight beta release of RGB version 0.11.

Beta 8 comes just three weeks after Beta 7, and mainly finalizes on work which has started before,
and contains bugfixes for the issues known before Beta 7 or discovered recently.

With Beta 8 RGB comes closer to API stabilization: we announce a freeze to any changes in API unless
they are required by a fix of a security-critical bug, putting at risk client's funds. All other
bugs and feature requests from now on will be deferred to v0.11.x maintenance versions, after the
completed v0.11.0 release cycle.

All users of RGB protocol, developers and integrators are urged to update and test their software
with Beta 8 as soon as possible. Due to the consensus-level changes contracts issued before this
release would not be compatible with the released RGB version.


## What's new in Beta 8

### Consensus-level change

Beta 8 amends implementation of [RCP-240731A], a new consensus-level feature we introduced in beta 7.
Specifically, it changes the type of `nonce` from `u8` to `u64`, in order to be able to support
**Lightning eltoo** and other protocols which may have many more offchain state updates as a part
of one transaction graph comparing to the modern-day Lightning BOLT approach.

### Improved persistence

In Beta 7 we have introduced an improved persistence APIs. In Beta 8 we complete this epic by
separating persistence providers from the data, such that users of RGB and BP libraries can now
integrate blocking storage code into async apps by using dedicated threads abstracted away through
these providers.


## What's next

Beta 8 release continues a phase of preparation for a public preview of the RGB v0.11. All parties
are strongly recommended switching to this latest beta, which contains a number of bugfixes and a
consensus-breaking change, and provide us feedback their feedback.

Beta 8 starts a period of codebase freeze: only critical vulnerability changes will be accepted to
the codebase. Once there were no critical vulnerabilities discovered, a Preview 1 version of RGB
will be released.

Beta 8 was a result of internal audit and testing effort, we do together with [Bitfinex Labs], which
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
