+++
title = "RGB v0.11 Beta 5 is out"
date = 2024-03-29
authors = ["Maxim Orlovsky"]
description = "A new beta release of Bitcoin and Lightning smart contract protocol"
+++

LNP/BP Standards Association is happy to announce the fifth beta release of RGB protocol version 0.11.

This is another milestone on the stabilization of the new v0.11 RGB version. It is the first
complete beta release of both consensus-level libraries, standard libraries and tools happening
in 2024, opening a way for independent developers integrating with RGB to test all the new
functionality.


## What's new in Beta 5

In preparing Beta 5 release we were focusing on performing internal audit of all consensus-critical
code, ensuring both its security and ability to do a protocol upgrades in the future versions
without introducing any incompatibilities with the assets and software using v0.11.

The internal audit was performed by two teams: the dev team and documentation team. The developer
team was reviewing the source code and modelling different types of attacks; while the doc team
was independently creating formal specification explaining how the protocol actually works (basing
on the source code), which was later compared to the expected protocol behaviour.

Let's walk through the major areas of the improvements shipping in v0.11 beta 5.

### Commitments

Cryptographic commitments, which go into Bitcoin blockchain in hidden form, are the core of RGB 
security and anti-double-spending protection. They are a part of so-called “single-use seals”, 
which you may have heard of before.

During the last three months (since Beta 3) we have passed through an extensive internal audit 
of the commitment procedures and wrote a lot of tests. We have improved the security and safety 
of many aspects of them and made sure they will be a proper foundation for the RGB.

One of the main changes which the end-users will notice is that now contracts (and assets) will 
commit to the issuer identity, simplifying the verification by the wallets and preventing scam 
attacks.

### Contractum language and scripting

With v0.11 one of our main focuses was simplification of RGB developer experiences.

First, we have introduced more opcodes allowing scripts in RGB contracts to access the contract 
state and use that information in validation. We have also developed RGB assembly, which can be 
compiled using macro `rgbasm` directive right from the Rust!

Finally, we are moving fast in finalizing the initial Contractum version: the language intended 
for RGB smart contract developers. In this release we add a Contractum representation for all 
standard interfaces used by RGB (RGB20 fungible assets, RGB21 NFTs and RGB25 fungible collections).

### Wallet functionality

With v0.11 RGB is integrated with wallet functionality out of the box. This solves one of the 
major dev and user headaches.

With Beta 5, we are going even further: now the command-line tool and RGB runtime can filter and 
present the information specific to a wallet of user choice and ignore the rest of the contract 
state.

Another new feature is the support for legacy Electrum RPC API, in addition to Esplora REST APIs, 
supported before.

### Containers

In this beta we have refactored the concept of containers. Containers are they way user exchange 
RGB data, for instance when they do distribute assets, perform transfers, -- or when developers 
distribute new code and interfaces for asset issuers.

In Beta 5 we have improved ASCII armoring of container data, so all of them now use more compact 
Base85 encoding and standard set of headers. We have simplified programming API for working with 
containers, and made sure their IDs are uniquely reflect their content - such that even the same 
contract disclosing different parts of information will have a distinct id.

We also simplified a life of devs and testers by providing an ability to convert any container 
into a human-readable YAML representation, edit it and convert back into a binary form, which 
can go into RGB system. This will simplify audit and debugging, allowing us to model different 
kinds of attacks on the system.


## About RGB v0.11

RGB v0.11 is an evolution of the protocol coming with a lot of bug fixes and improvements, further
enhancing RGB smart contracting capabilities. It will be the first version which will be audited by
independent auditors, after which it can be considered by third-party issuers for the use in
production.

The main features shipping in v0.11 will be:
- Wallet integration right into RGB runtime and command-line tool;
- Basic support for Liquid sidechain, with ability to add more alternative scalability layers in the 
  future;
- Scripting, with new state introspection codes and RGB assembly compiler;
- Initial support for Contractum language;
- Ability to inherit interfaces (multiple inheritance!);
- Support of Electrum RPC additionally to Esplora REST, present before;
- Commitments to issuer and developer identities embedded into contracts, interfaces, libraries;
- More compact consignments and better ASCII armoring.


## What's next

With Beta 5 release RGB development enters a new phase of preparation for a public preview of 
the RGB v0.11. The public preview will be a feature-complete version which will be used by 
external auditors for evaluating the safety of the protocol. At the same time app devs and users 
will be able to complete integration of RGB without expecting major API or consensus break with 
the release.

Once there will be results of an audit, the preview version will become an official release (if 
the audit will find any major issues however, they will be fixed even if the fixes will contain 
a breaking changes).

You can track us on our journey towards v0.11 release with this [GitHub dashboard][proj].


## Acknowledgements

We are grateful to [Fulgur Ventures], providing another year of support for our efforts in 
developing RGB. This year they were joined by [Bitlight Labs], who had become a new full member of 
the Association.

We have received a number of important contributions and bugfixes coming from commercial 
companies, such as [Pandora Prime], [Bitfinex] and [DIBA]. We are also grateful for individual 
contributors, who do their small -- but still highly valuable and welcomed input in making RGB 
better.


[proj]: https://github.com/orgs/RGB-WG/projects/17/views/1
[Fulgur Ventures]: https://fulgur.ventures
[Bitlight Labs]: https://bitlightlabs.com
[Pandora Prime]: https://pandoraprime.ch
[Bitfinex]: https://www.bitfinex.com/about
[DIBA]: http://diba.io
