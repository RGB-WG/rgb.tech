+++
title = "API references"
weight = 30
[extra]
anchor = "api"
multicolumn = true
bg-color = "white"
+++

### Consensus level APIs

| Library          | Description                                                                           | API Reference                      | Source code                                                                                              |
|------------------|---------------------------------------------------------------------------------------|------------------------------------|----------------------------------------------------------------------------------------------------------|
| Commit-verify    | Commit-verify cryptographic schemes and multi-protocol commitments (LNPBP-4 standard) | <https://docs.rs/commit_verify>    | [LNP-BP/commit_verify](https://github.com/LNP-BP/client_side_validation/tree/master/commit_verify)       |
| Single-use-seals | Single-use-seals abstract APIs                                                        | <https://docs.rs/single_use_seals> | [LNP-BP/single_use_seals](https://github.com/LNP-BP/client_side_validation/tree/master/single_use_seals) |
| BP DBC           | Deterministic bitcoin commitments on bitcoin protocol                                 | <https://docs.rs/bp-dbc>           | [BP-WG/bp-dbc](https://github.com/BP-WG/bp-core/tree/master/dbc)                                         | 
| BP Seals         | Single-use-seals with bitcoin protocol                                                | <https://docs.rs/bp-seals>         | [BP-WG/bp-seals](https://github.com/BP-WG/bp-core/tree/master/seals)                                     | 
| RGB Core         | Main consensus code for RGB                                                           | <https://docs.rs/rgb-core>         | [RGB-WG/rgb-core](https://github.com/RGB-WG/rgb-core)                                                    |

### Integration level APIs

| Library          | Description                                                                                   | API Reference                   | Source code                                                            |
|------------------|-----------------------------------------------------------------------------------------------|---------------------------------|------------------------------------------------------------------------|
| RGB Standard Lib | RGB standard library for working with smart contracts on the lowest level                     | <https://docs.rs/rgb-std>       | [RGB-WG/rgb-std](https://github.com/RGB-WG/rgb-wallet/tree/master/std) |
| RGB Wallet Lib   | Wallet-integration library for WASM apps providing RGB invoices and PSBT functionality        | <https://docs.rs/rgb-wallet>    | [RGB-WG/rgb-wallet](https://github.com/RGB-WG/rgb-wallet)              |
| RGB Lib          | High-level RGB integration library for mobile & desktop apps providing FS-based stash storage | <https://docs.rs/rgb-contracts> | [RGB-WG/rgb](https://github.com/RGB-WG/rgb)                            |
