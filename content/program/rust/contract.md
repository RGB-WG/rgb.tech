+++
weight = 10
title = "Writing simple contract"
[extra]
anchor = "basics"
+++

Firstly, using cargo to create a new project: `cargo new rgb20-token`, and enter
it: `cd rgb20-token`. Then create a directory to store the contract file. You may
name it whatever you like, here we set it to `examples`: `mkdir examples`.
See source codes at [RGB20 Contract](https://github.com/bitlightlabs/bitlight-rgb20-contract).

Here is the complete project file tree:

```
➜  rgb20-token git:(main) ✗ tree .
.
├── Cargo.lock
├── Cargo.toml
├── contracts
└── src
    └── main.rs

3 directories, 3 files
```

Secondly, we add some libs in `Cargo.toml` file:

```toml
[package]
name = "rgb20-token"
version = "0.1.0"
edition = "2021"
resolver = "2"

[dependencies]
amplify = "4.6.0"
ascii-armor = "0.2.0"
strict_encoding = "2.7.0-beta.1"
strict_types = "2.7.0-beta.1"
aluvm = { version = "0.11.0-beta.4", features = ["log"] }
bp-core = "0.11.0-beta.4"
rgb-std = { version = "0.11.0-beta.4", features = ["serde", "fs"] }
serde = "1.0"
serde_json = "1.0"
sha2 = "0.10.8"
rgb-schemata = "0.11.0-beta.4"
hex = "0.4.3"

[dev-dependencies]
chrono = "0.4.31"
serde_yaml = "0.9.27"

[features]
all = []

[patch.crates-io]
commit_verify = { git = "https://github.com/LNP-BP/client_side_validation", branch = "master" }
bp-consensus = { git = "https://github.com/BP-WG/bp-core", branch = "master" }
bp-dbc = { git = "https://github.com/BP-WG/bp-core", branch = "master" }
bp-seals = { git = "https://github.com/BP-WG/bp-core", branch = "master" }
bp-core = { git = "https://github.com/BP-WG/bp-core", branch = "master" }
rgb-core = { git = "https://github.com/RGB-WG/rgb-core", branch = "master" }
rgb-std = { git = "https://github.com/RGB-WG/rgb-std", branch = "master" }
rgb-schemata = { git = "https://github.com/RGB-WG/rgb-schemata", branch = "master" }
```

Then, in `src/main.rs`, write following codes:

```rust
use std::convert::Infallible;
use std::fs;

use amplify::hex::FromHex;
use armor::AsciiArmor;
use bp::dbc::Method;
use bp::{Outpoint, Txid};
use rgb_schemata::NonInflatableAsset;
use rgbstd::containers::FileContent;
use rgbstd::interface::{FilterIncludeAll, FungibleAllocation, IfaceClass, IssuerClass, Rgb20};
use rgbstd::invoice::Precision;
use rgbstd::persistence::{Inventory, Stock};
use rgbstd::resolvers::ResolveHeight;
use rgbstd::validation::{ResolveWitness, WitnessResolverError};
use rgbstd::{WitnessAnchor, WitnessId, XAnchor, XPubWitness};
use strict_encoding::StrictDumb;

struct DumbResolver;

impl ResolveWitness for DumbResolver {
    fn resolve_pub_witness(&self, _: WitnessId) -> Result<XPubWitness, WitnessResolverError> {
        Ok(XPubWitness::strict_dumb())
    }
}

impl ResolveHeight for DumbResolver {
    type Error = Infallible;
    fn resolve_anchor(&mut self, _: &XAnchor) -> Result<WitnessAnchor, Self::Error> {
        Ok(WitnessAnchor::strict_dumb())
    }
}

#[rustfmt::skip]
fn main() {
    let beneficiary_txid =
        Txid::from_hex("d6afd1233f2c3a7228ae2f07d64b2091db0d66f2e8ef169cf01217617f51b8fb").unwrap();
    let beneficiary = Outpoint::new(beneficiary_txid, 1);

    let contract = NonInflatableAsset::testnet("TEST", "Test asset", None, Precision::CentiMicro)
        .expect("invalid contract data")
        .allocate(Method::TapretFirst, beneficiary, 100_000_000_000_u64.into())
        .expect("invalid allocations")
        .issue_contract()
        .expect("invalid contract data");

    let contract_id = contract.contract_id();

    eprintln!("{contract}");
    contract.save_file("examples/rgb20-simplest.rgb").expect("unable to save contract");
    fs::write("examples/rgb20-simplest.rgba", contract.to_ascii_armored_string()).expect("unable to save contract");

    // Let's create some stock - an in-memory stash and inventory around it:
    let mut stock = Stock::default();
    stock.import_iface(Rgb20::iface()).unwrap();
    stock.import_schema(NonInflatableAsset::schema()).unwrap();
    stock.import_iface_impl(NonInflatableAsset::issue_impl()).unwrap();

    stock.import_contract(contract, &mut DumbResolver).unwrap();

    // Reading contract state through the interface from the stock:
    let contract = stock.contract_iface_id(contract_id, Rgb20::iface().iface_id()).unwrap();
    let contract = Rgb20::from(contract);
    let allocations = contract.fungible("assetOwner", &FilterIncludeAll).unwrap();
    eprintln!("\nThe issued contract data:");
    eprintln!("{}", serde_json::to_string(&contract.spec()).unwrap());

    for FungibleAllocation  { seal, state, witness, .. } in allocations {
        eprintln!("amount={state}, owner={seal}, witness={witness}");
    }
    eprintln!("totalSupply={}", contract.total_supply());
    eprintln!("created={}", contract.created().to_local().unwrap());
}


```

Save it, and execute `cargo run`, after that, contract file would save in
`examples` directory. You can specify your own token name, decimal,
description, beneficiary and supply by modifing corresponding variable's value,
as well as contracts saving fold.

Now, you can import the contract with `rgb import` command:
`rgb import examples/rgb20-simplest.rgb`.
