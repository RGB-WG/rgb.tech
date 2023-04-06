+++
weight = 10
title = "Writing simple contract"
[extra]
anchor = "basics"
+++

```rust
    let nominal = Nominal::new("TEST", "Test asset", Precision::CentiMicro);
    let contract_text = ContractText::default();
    let beneficiary = Outpoint::new(
        Txid::from_hex("623554ac1dcd15496c105a27042c438921f2a82873579be88e74d7ef559a3d91").unwrap(), 
        0
    );

    let contract = ContractBuilder::with(
        rgb20(),
        schema(),
        iface_impl()
        ).expect("schema fails to implement RGB20 interface")

        .set_chain(Chain::Testnet3)

        .add_global_state("Nominal", nominal)
        .expect("invalid nominal")

        .add_global_state("ContractText", contract_text)
        .expect("invalid contract text")

        .add_fungible_state("Assets", beneficiary, 1_000_000_0000_0000)
        .expect("invalid asset amount")

        .issue_contract()
        .expect("contract doesn't fit schema requirements");

    let contract_id = contract.contract_id();

    let bindle = contract.bindle();
    bindle.save("examples/rgb20-simplest.contract.rgb").expect("unable to save contract");
```
