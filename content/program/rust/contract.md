+++
weight = 10
title = "Writing simple contract"
[extra]
anchor = "basics"
+++

Firstly, using cargo to create a new project: `cargo new rgb20-usdt`, and enter
it: `cd rgb20-usdt`. Then create a directory to store the contract file. You may
name it whatever you like, here we set it to `contracts`: `mkdir contracts`.
See source codes at [RGB20-USDT](https://github.com/oneforalone/rgb20-usdt).

Here is the complete project file tree:
```
➜  rgb20-usdt git:(main) ✗ tree .
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
[denpendencies]
amplify = "4.0.1"
strict_encoding = "2.5.0"
strict_types = "1.6.1"
bp-core = "0.10.9"
rgb-std = { version = "0.10.8", features = ["serde", "fs"] }
serde = "1.0.152"
serde_json = "1.0.93"
rgb-schemata = "0.10.0"
```

Then, in `src/main.rs`, write following codes:

```rust
use rgbstd::interface::{rgb20, ContractBuilder};

use std::convert::Infallible;
use std::fs;

use amplify::hex::FromHex;
use bp::{Chain, Outpoint, Tx, Txid};
use rgb_schemata::{nia_rgb20, nia_schema};
use rgbstd::containers::BindleContent;
use rgbstd::contract::WitnessOrd;
use rgbstd::resolvers::ResolveHeight;
use rgbstd::stl::{
    Amount, ContractData, DivisibleAssetSpec, Precision, RicardianContract, Timestamp,
};
use rgbstd::validation::{ResolveTx, TxResolverError};
use strict_encoding::StrictDumb;

struct DumbResolver;

impl ResolveTx for DumbResolver {
    fn resolve_tx(&self, _txid: Txid) -> Result<Tx, TxResolverError> {
        Ok(Tx::strict_dumb())
    }
}

impl ResolveHeight for DumbResolver {
    type Error = Infallible;
    fn resolve_height(&mut self, _txid: Txid) -> Result<WitnessOrd, Self::Error> {
        Ok(WitnessOrd::OffChain)
    }
}

#[rustfmt::skip]
fn main() {
    let name = "USDT";  // name of token
    let decimal = Precision::CentiMicro; // Decimal: 8
    let desc = "USD Tether Token";  // token description
    let spec = DivisibleAssetSpec::with("RGB20", name, decimal, Some(desc)).unwrap();
    let terms = RicardianContract::default();
    let contract_data = ContractData { terms, media: None };
    let created = Timestamp::now();
    let txid = "9fef7f1c9cd1dd6b9ac5c0a050103ad874346d4fdb7bcf954de2dfe64dd2ce05";
    // token owner address in utxo
    let beneficiary = Outpoint::new(Txid::from_hex(txid).unwrap(), 0);

    const ISSUE: u64 = 1_000_000_000;

    let contract = ContractBuilder::with(rgb20(), nia_schema(), nia_rgb20())
        .expect("schema fails to implement RGB20 interface")
        .set_chain(Chain::Testnet3)
        .add_global_state("spec", spec)
        .expect("invalid nominal")
        .add_global_state("created", created)
        .expect("invalid creation date")
        .add_global_state("issuedSupply", Amount::from(ISSUE))
        .expect("invalid issued supply")
        .add_global_state("data", contract_data)
        .expect("invalid contract text")
        .add_fungible_state("assetOwner", beneficiary, ISSUE)
        .expect("invalid asset amount")
        .issue_contract()
        .expect("contract doesn't fit schema requirements");

    let contract_id = contract.contract_id();
    debug_assert_eq!(contract_id, contract.contract_id());

    let bindle = contract.bindle();
    eprintln!("{bindle}");
    bindle.save("contracts/rgb20-usdt.contract.rgb")
        .expect("unable to save contract");
    fs::write("contracts/rgb20-usdt.contract.rgba", bindle.to_string())
        .expect("unable to save contract");
}

```

Save it, and execute `cargo run`, after that, contract file would save in
`contracts` directory. You can specify your own token name, decimal,
description, beneficiary and supply by modifing corresponding variable's value,
as well as contracts saving fold.

Now, you can import the contract with `rgb import` command:
`rgb import contracts/rgb20-usdt.contract.rgb`.
