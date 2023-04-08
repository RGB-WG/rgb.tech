+++
weight = 10
title = "Overview of RGB development"
[extra]
bg-color = "green"
+++

Contract business logic in RGB is defined through so-called *schema*; the 
specific RGB contract must implement some *schema*. One may think of a schema 
as a "class" definition in the OOP world; in that terms, specific RGB contracts 
are "class instances" created by the schema constructor (*genesis operation*). 
Such approach allows to separate role of contract developer (in RGB called 
*schema developer*) and *contract issuer*: the first one needs to be an 
experienced developer, while the second one is not required to know anything 
about coding or security at all. This also promotes re-use of common codebase by
different issuers for the same typical use cases (like fungible assets), 
reducing the risk of mistakes.

RGB uses specially-designed virtual machine [AluVM], which is Turing-complete in
the same terms as EVM and WASM-based smart contracts (i.e. nearly computational
universal, but is bound by number of operation steps, measured by gas 
consumption in Ethereum-like systems, and by accumulated computational 
complexity measure in case of AluVM).

[AluVM]: https://www.aluvm.org/
[strict types]: https://www.strict-types.org/