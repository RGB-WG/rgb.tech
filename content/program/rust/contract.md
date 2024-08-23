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
strict_encoding = "2.7.0-beta.4"
bp-core = "0.11.0-beta.6"
rgb-std = { version = "0.11.0-beta.6", features = ["serde", "fs"] }
serde = "1.0"
serde_json = "1.0"
rgb-schemata = "0.11.0-beta.6"
rgb-interfaces = "0.11.0-beta.6"
bdk = { version = "1.0.0-alpha.11", default-features = false }
bdk_esplora = { version = "0.14.0", features = ["blocking"] }
hex = "0.4.3"

[features]
all = []

```

Then, in `src/main.rs`, write following codes:

```rust
use amplify::hex::FromHex;
use bp::dbc::Method;
use bp::{Outpoint, Txid};
use ifaces::Rgb20;
use rgbstd::containers::{FileContent, Kit};
use rgbstd::interface::{FilterIncludeAll, FungibleAllocation};
use rgbstd::invoice::Precision;
use rgbstd::persistence::{MemIndex, MemStash, MemState, Stock};
use schemata::dumb::DumbResolver;
use schemata::NonInflatableAsset;

#[rustfmt::skip]
fn main() {
    let beneficiary_txid = 
        Txid::from_hex("311ec7d43f0f33cda5a0c515a737b5e0bbce3896e6eb32e67db0e868a58f4150").unwrap();
    let beneficiary = Outpoint::new(beneficiary_txid, 1);

    let contract = Rgb20::testnet::<NonInflatableAsset>("ssi:anonymous","TEST", "Test asset", None, Precision::CentiMicro)
        .expect("invalid contract data")
        .allocate(Method::TapretFirst, beneficiary, 100_000_000_000_u64)
        .expect("invalid allocations")
        .issue_contract()
        .expect("invalid contract data");

    let contract_id = contract.contract_id();

    eprintln!("{contract}");
    contract.save_file("examples/rgb20-simplest.rgb").expect("unable to save contract");
    contract.save_armored("examples/rgb20-simplest.rgba").expect("unable to save armored contract");

    let kit = Kit::load_file("schemata/NonInflatableAssets.rgb").unwrap().validate().unwrap();

    // Let's create some stock - an in-memory stash and inventory around it:
    let mut stock = Stock::<MemStash, MemState, MemIndex>::default();
    stock.import_kit(kit).expect("invalid issuer kit");
    stock.import_contract(contract, &mut DumbResolver).unwrap();

    // Reading contract state through the interface from the stock:
    let contract = stock.contract_iface_class::<Rgb20>(contract_id).unwrap();
    // let contract = Rgb20::from(contract);
    let allocations = contract.fungible("assetOwner", &FilterIncludeAll).unwrap();
    eprintln!("\nThe issued contract data:");
    eprintln!("{}", serde_json::to_string(&contract.spec()).unwrap());

    for FungibleAllocation  { seal, state, witness, .. } in allocations {
        eprintln!("amount={state}, owner={seal}, witness={witness}");
    }
    eprintln!("totalSupply={}", contract.total_supply());
}


```

Save it, and execute `cargo run`, after that, contract file would save in
`examples` directory. You can specify your own token name, decimal,
description, beneficiary and supply by modifying corresponding variable's value,
as well as contracts saving fold.

Now, you can import the contract with `rgb import` command:
`rgb import examples/rgb20-simplest.rgb`.
