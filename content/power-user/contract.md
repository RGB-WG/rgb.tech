+++
title = "Using contracts"
weight = 10
[extra]
bg-color = "green"
anchor = "contract"
+++

A contract is distributed as a package, named **contract consignment**.

*Consignment* is a term in RGB denoting a package of client-side-validated
data passed between peers. Consignments may be of two different types:
*contract consignments*, which distribute public contract information from a
contract issuer to users, and *transfer consignments*, which are used for doing
state transfers between contract users. In this chapter we will address the
first type of consignments, leaving the second for a next chapter.

RGB stores all data required for its operations in a storage called **stash**.
Since RGB is not a blockchain-based smart contracts, the stash is not replicated
to other peer machines and usually keeps the data which nobody else than you
know or have access to (at least to some of those data). Thus, losing stash
means loosing your assets and contract state, and it has to be backed up with
the same effort as a seed phrase or wallet descriptor for a complex wallet
setup.

To import a contract into your stash one first needs to get a contract
consignment from a contract issuer. A contract consignment is either a binary
file – or an ASCII/Base64-armoded text (and example can be found it the
["issuing contract" section](/power-user/#issue)).

```sh
$ rgb import contract.rgb
```

To list contracts which are part of the stash you need to run `rgb` commands
with no arguments provided (or `rgb info` as an alternative to this)

```sh
$ rgb
```

Which should output something like

```
Schemata:
---------
marina-karl-basic-4h3xkAmiRdQs2HwQmnFVqoj5DG3NhGkJRpNZXjNrJT2N: RGB20

Interfaces:
---------
RGB20 complex-field-union-DbwzvSu4BZU81jEpE9FVZ3xjcyuTKWWy2gmdnaxtACrS

Contracts:
---------
rgb:BasilYellowGrand0BgHH3yYaCUvaw5Tc8KfrVB1aWN4vBvCXCuMfK98N3o6Z
```

Here you see your imported contract id, plus a schema, used by this contract
which implements RGB20 interface. Now, you can read the state of the contract
through this interface:

```shell
$ rgb state BasilYellowGrand0BgHH3yYaCUvaw5Tc8KfrVB1aWN4vBvCXCuMfK98N3o6Z RGB20
```

```
Global state:
Nominal:=(ticker=("TEST"), name=("Test asset"), details=~, precision=8)

Owned state:
  (amount=100000000000000, owner=913d9a55efd7748ee89b577328a8f22189432c04275a106c4915cd1dac543562:0, witness=~)
```

Here you see that the contract has both global state, representing the
information about RGB20 token, and an owned state – certain amount of that token
assigned to a UTXO.
