+++
title = "RGB v0.12 release candidate 2"
date = 2025-06-01
authors = ["Maxim Orlovsky", "Olga Ukolova"]
description = "RGB 0.12 reaches second release candidate, getting closer to the production"
+++

LNP/BP Standards Association presented a new generation of RGB smart contracts (version 0.12)
[10 days ago with release candidate 1][RC1]. Over this period of time we have received several
improvement requests from the companies already integrating it into their products and tried
to fulfill them with release candidate 2.

Alongside candidate 2 we release the updated version of fungible and non-fungible token standards,
namely [RGB-20] and [RGB-21].

What's new
----------

### Consensus-level

This release introduces significant enhancements to the Ultrasonic contract execution layer,
focusing on improved security, flexibility, and code organization.
The primary goal is to enable more complex and secure contract logic by providing mechanisms for
input-specific lock conditions and operation-level witnesses.

- **Codex Structure Modification:** The `Codex` struct is updated to include a `features` field for
  enabling/disabling certain functionalities.
  This allows for future extensibility and configuration options.
- **Two-Phase Verification:** The contract verification process is split into two distinct phases:
  a main verification script and input-specific lock script verification.
  The main verification script can now focus on the overall operation logic,
  while the lock scripts can handle input-specific access control.
  This allows for more granular control over input access conditions.
- **CellLock Structure:** A new `CellLock` struct is introduced to encapsulate the lock script and
  auxiliary data associated with a destructible state cell,
  which provides a cleaner and more structured way to manage lock conditions.
- **Operation Witness:** An operation-level `witness` field is added to the `Operation` and `Genesis` structs,
  allowing for data that applies to the entire operation, rather than just individual inputs.
- **Ultrasonic Instruction Set Expansion:**
  New instructions (`LdW`, `LdIW`, `LdIL`, `LdIT`) are added to the Ultrasonic instruction set to
  facilitate access to the operation witness, input-specific witness, lock auxiliary data,
  and authorization token within the VM.

### Standard library

This release refactors the RGB smart contracts library to improve its persistence layer,
enhance security, and streamline consignment handling.

- Replaced the `fs` feature with a `binfile` feature, providing more granular control over binary file format support.
- Introduced a new `rgb-persist-fs` crate to encapsulate filesystem persistence logic,
  improving code modularity and separation of concerns.
- Introduced the way to programmatically import contracts via consignments.
- Improved spam prevention, which included creation of a dedicated guideline to contract developers ([RCP-250528A]),
  its implementation, and the addition of an embedded mechanism limiting the maximal number of operations in a consignment.
- Added more aggregators producing computed contract state.
- Replaced `ImmutableApi` and `DestructibleApi` with `GlobalApi` and `OwnedApi`, respectively, for clearer semantics.
- Introduced `Semantics` struct to group API definitions, codex libraries, and type systems.
- Added `ApisChecksum` and `ArticlesId` for improved API versioning and identification.
- Implemented signature validation in `Articles::with` and `Issuer::with` using a provided validator function.
- The test suite has been updated to include more comprehensive validation of the new API
  and consignment handling mechanisms.

### Runtime and command-line

Runtime and the `rgb` command-line tool has got complete implementation for:
- Contract import and export;
- Contract backups;
- Purging of non-used contracts.


### Interface standards

Release candidate 2 includes the updated version of fungible and non-fungible token standards,
namely [RGB-20] and [RGB-21].


Using the release
-----------------

The source code of the release is available on [GitHub](https://github.com/RGB-WG/rgb/tree/v0.12.0-rc.2).

Those power users who'd like to play with the command line
may install it either from the sourcecode or using Rust Cargo command

```bash
cargo install rgb-wallet --version 0.12.0-rc.2
```

Demo & examples can be found here:
- https://github.com/RGB-WG/rgb/tree/v0.12.0-rc.2/examples
- https://github.com/RGB-WG/rgb-sandbox/blob/v0.12/demo.sh
- https://github.com/pandora-prime/rgb-issuers/

To start integrating RGB, please feel free to contract LNP/BP Standards Association
via e-mail info at lnp-bp dot org.



[RC1]: https://rgb.tech/blog/release-v0-12-rc-1/
[RGB-20]: https://github.com/RGB-WG/RFC/blob/master/RGB-20.md
[RGB-21]: https://github.com/RGB-WG/RFC/blob/master/RGB-21.md
[RCP-250528A]: https://github.com/RGB-WG/RFC/issues/18
