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
amplify = "4.7.0"
strict_encoding = "2.7.0-rc.1"
bp-core = "0.11.0-beta.7"
rgb-std = { version = "0.11.0-beta.7", features = ["serde", "fs"] }
rgb-schemata = "0.11.0-beta.7"
rgb-interfaces = "0.11.0-beta.7"
serde = "1.0"
serde_json = "1.0"

[features]
all = []
```

Then, in `src/main.rs`, write following codes:

```rust
use amplify::hex::FromHex;
use bp::dbc::Method;
use bp::{Outpoint, Txid};
use ifaces::Rgb20;
use rgbstd::containers::{ConsignmentExt, FileContent};
use rgbstd::interface::{FilterIncludeAll, FungibleAllocation};
use rgbstd::invoice::Precision;
use rgbstd::persistence::Stock;
use rgbstd::XWitnessId;
use schemata::dumb::NoResolver;
use schemata::NonInflatableAsset;

#[rustfmt::skip]
fn main() {
    let beneficiary_txid = 
        Txid::from_hex("311ec7d43f0f33cda5a0c515a737b5e0bbce3896e6eb32e67db0e868a58f4150").unwrap();
    let beneficiary = Outpoint::new(beneficiary_txid, 1);

    #[allow(clippy::inconsistent_digit_grouping)]
    let contract = NonInflatableAsset::testnet("ssi:anonymous","TEST", "Test asset", None, Precision::CentiMicro, [(Method::TapretFirst, beneficiary, 1_000_000_000_00u64)])
        .expect("invalid contract data");

    let contract_id = contract.contract_id();

    eprintln!("{contract}");
    contract.save_file("examples/rgb20-simplest.rgb").expect("unable to save contract");
    contract.save_armored("examples/rgb20-simplest.rgba").expect("unable to save armored contract");

    // Let's create some stock - an in-memory stash and inventory around it:
    let mut stock = Stock::in_memory();
    stock.import_contract(contract, NoResolver).unwrap();

    // Reading contract state through the interface from the stock:
    let contract = stock.contract_iface_class::<Rgb20>(contract_id).unwrap();
    let allocations = contract.allocations(&FilterIncludeAll);
    eprintln!("\nThe issued contract data:");
    eprintln!("{}", serde_json::to_string(&contract.spec()).unwrap());

    for FungibleAllocation  { seal, state, witness, .. } in allocations {
        let witness = witness.as_ref().map(XWitnessId::to_string).unwrap_or("~".to_owned());
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
