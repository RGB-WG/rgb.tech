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
with no arguments provided (or `rgb contracts` as an alternative to this)

```sh
$ rgb contracts
```

Which should output something like

```
rgb:D4RN7r4$-ZNt43c$-ymINZ1r-M$bJPPf-SWp9193-OLIdtv0	bitcoin            	2024-09-07	rgb:sch:RDYhMTR!9gv8Y2GLv9UNBEK1hcrCmdLDFk9Qd5fnO8k#brave-dinner-banana         
  Developer: ssi:anonymous
```

Here you see your imported contract id, plus a schema, used by this contract
which implements RGB20 interface. Now, you can read the state of the contract
through this interface:

```shell
$ rgb state 'D4RN7r4$-ZNt43c$-ymINZ1r-M$bJPPf-SWp9193-OLIdtv0' RGB20Fixed
```

```
Global:
  spec := (ticker=("DBG"), name=("Debug asset"), details=1(("Pay attention: the asset has no value")), precision=2)
  terms := (text=("SUBJECT TO, AND WITHOUT IN ANY WAY LIMITING, THE REPRESENTATIONS AND WARRANTIES OF ANY SELLER  EXPRESSLY SET FORTH IN THIS AGREEMENT OR ANY OTHER EXPRESS OBLIGATION OF SELLERS PURSUANT TO THE TERMS HEREOF, AND ACKNOWLEDGING THE PRIOR USE OF THE PROPERTY AND PURCHASER’S OPPORTUNITY  TO INSPECT THE PROPERTY, PURCHASER AGREES TO PURCHASE THE PROPERTY “AS IS”, “WHERE IS”,  WITH ALL FAULTS AND CONDITIONS THEREON. ANY WRITTEN OR ORAL INFORMATION, REPORTS, STATEMENTS,  DOCUMENTS OR RECORDS CONCERNING THE PROPERTY PROVIDED OR MADE AVAILABLE TO PURCHASER, ITS AGENTS OR CONSTITUENTS BY ANY SELLER, ANY SELLER’S AGENTS, EMPLOYEES OR THIRD PARTIES REPRESENTING OR PURPORTING TO REPRESENT ANY SELLER, SHALL NOT BE REPRESENTATIONS OR WARRANTIES, UNLESS SPECIFICALLY SET FORTH HEREIN. IN PURCHASING THE PROPERTY OR TAKING OTHER ACTION HEREUNDER, PURCHASER HAS NOT AND SHALL NOT RELY ON ANY SUCH DISCLOSURES, BUT RATHER, PURCHASER SHALL RELY ONLY ON PURCHASER’S OWN INSPECTION OF THE PROPERTY AND THE REPRESENTATIONS AND WARRANTIES  HEREIN. PURCHASER ACKNOWLEDGES THAT THE PURCHASE PRICE REFLECTS AND TAKES INTO ACCOUNT THAT THE PROPERTY IS BEING SOLD “AS IS”.
"), media=~)
  issuedSupply := (100000000)

Owned:
  assetOwner:
    value=100000000, utxo=bc:tapret1st:b449f7eaa3f98c145b27ad0eeb7b5679ceb567faef7a52479bc995792b65f804:1, witness=~ -- owner unknown
```

Here you see that the contract has both global state, representing the
information about RGB20 token, and an owned state – certain amount of that
token assigned to an UTXO. Since we do not use any wallet, it shows that the
owner of this output is not known.
