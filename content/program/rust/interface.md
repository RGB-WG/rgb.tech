+++
weight = 50
title = "Doing custom interface"
[extra]
anchor = "interface"
+++

```rust
pub fn rgb20() -> Iface {
    let types = StandardTypes::new();

    Iface {
        name: tn!("RGB20"),
        global_state: tiny_bmap! {
            tn!("Nominal") => Req::require(types.get("RGBContract.Nominal")),
            tn!("ContractText") => Req::require(types.get("RGBContract.ContractText")),
        },
        assignments: tiny_bmap! {
            tn!("Assets") => AssignIface::private(OwnedIface::Amount),
        },
        valencies: none!(),
        genesis: GenesisIface {
            metadata: None,
            global: tiny_bmap! {
                tn!("Nominal") => Occurrences::Once,
                tn!("ContractText") => Occurrences::Once,
            },
            assignments: tiny_bmap! {
                tn!("Assets") => Occurrences::OnceOrMore
            },
            valencies: none!(),
        },
        transitions: tiny_bmap! {
            tn!("Transfer") => TransitionIface {
                metadata: None,
                globals: none!(),
                inputs: tiny_bmap! {
                    tn!("Assets") => Occurrences::OnceOrMore,
                },
                assignments: tiny_bmap! {
                    tn!("Assets") => Occurrences::OnceOrMore,
                },
                valencies: none!(),
                default_assignment: Some(tn!("Assets")),
            }
        },
        extensions: none!(),
        default_operation: Some(tn!("Transfer")),
    }
}
```
