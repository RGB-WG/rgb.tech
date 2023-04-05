+++
weight = 20
title = "Components"
[extra]
bg-color = "red"
multicolumn = true
+++

In RGB, there is no such thing as "program a smart contract"; instead, contracts
themselves are issued with no conde. This helps to simplify life of asset 
issuers and normies, who just need to issue a token, create their self-sovereign
identity, run a DAO etc.

This is achieved through separation of concerns, where the development work is
kept outside contracts, thanks to things named **schema** and **interface**.

* ### Contract

    The contract is a contract genesis plus a history of operations on the
    contract. Together they are used by RGB to compute a deterministic contract
    state. To create a contract means to issue a contract genesis, which 
    requires no code at all. Contract issuer just needs to pick a **schema**
    which has all features he needs in a contract – and use it during the 
    issuance. If the schema with the required properties doesn't exist, issuer
    can hire a developer to implement such schema.

* ### Schema

    If you think in terms of objective-oriented programming (OOP), contract is
    an *instance* of some *class*, and in RGB terms this *class* is named
    **schema**. Schema contains all contract business logic, scripts, 
    definitions of state types (owned and global), allowed operations, rules
    for validation, data type system etc. Schema by its nature is declarative,
    this it can be easily read or written with rust macros, YAML, TOML and other
    declarative languages. Once created, schema may be used by many contracts
    of the same type.

* ### Interface

    An interface defines a way how user-facing software (wallets etc) can 
    interact with a contract. Interface provides **semantic** information about
    the contract and defines how its state can be presented to the user.
    If one can compare schema to a *class* in OOP, than **contract interface**
    should be compared to Java *interface* or Rust *trait*. As classes can
    implement interfaces in Java, or structs can implement traits in Rust,
    in RGB schema can implement many interfaces.
