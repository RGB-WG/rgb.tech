+++
weight = 20
title = "Creating custom state"
[extra]
anchor = "state"
+++

Custom state can be added to a contract by defining a data type which will hold 
it. Data type is a rust structure or enum, which implements `strict_encoding`
traits. The simplest way to add this implementation is through derive macros:

```rust
#[derive(Clone, Eq, PartialEq, Debug)]
#[derive(StrictDumb, StrictType, StrictEncode, StrictDecode)]
#[strict_type(lib = LIB_NAME_RGB_CONTRACT)]
#[cfg_attr(
    feature = "serde",
    derive(Serialize, Deserialize),
    serde(crate = "serde_crate", rename_all = "camelCase")
)]
pub struct Nominal {
    ticker: Ticker,
    name: ContractName,
    details: Option<ContractDetails>,
    precision: Precision,
}
impl StrictSerialize for Nominal {}
impl StrictDeserialize for Nominal {}
```

Once declared, the type can be compiled into a type library:

```rust
let lib = LibBuilder::new(libname!(LIB_NAME_RGB_CONTRACT))
    .process::<Nominal>()?
    .compile(none!())?;
let types = SystemBuilder::new()
    .import(lib)?
    .finalize()?;
```
