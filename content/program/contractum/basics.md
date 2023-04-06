+++
weight = 10
title = "Writing simple contract"
[extra]
bg-color = "white"
+++

The best way to understand RGB is through example. Let's do a contract which
will work as a digital identity.

We have to start with definition of a data type representing the key. This is
not a part of the contract itself and may be imported into multiple contracts
from a _data type library_.

```haskell
data PgpKey :: curve U8, key Bytes
```

As you might see, the code looks similar to Haskel. New data types are declared
with `data` keyword, and can be composed of named fields (being structs),
unnamed fields (tuples), operate as enums (in this case you should use
`|` instead of `,`), or unions, where each variant may in turn have an
associated structure or tuple. You may read more about constructing data types
in [strict types] guidelines.

Now, having the data type we can start definition of our schema:

```haskell
schema DecentralizedIdentity
   -- This defines the atom of the contract state called `Identity`
   -- which has data type `PgpKey`.
   -- The `owned` keyword means that there is always a party
   -- which owns the identity
   owned Identity :: PgpKey
   -- an implicit part of the story is the fact that owned state
   -- is always assigned to a single-use-seal definition, so the
   -- proper way of writing the above statement will be
   owned Identity :: SingleUseSeal -> PgpKey
   -- but we leave the part which repeats for each and every owned
   -- state to reduce boilerplate of writing `SingleUseSeal ->`
   -- over and over again.

   -- This says that to construct contract the user must provide
   -- information about exactly one identity
   genesis :: Identity

   -- Now let's define what a owner of identity can do,
   -- He can execute his rights by creating state transitions
   -- ("operation" on the state) of predefined forms, like
   op Revocation :: old Identity -> new Identity
   -- which does what it says: it revokes existing identity
   -- and creates a new one.
```

As you may see, schema declares state which the contract operates, and
operations, which may update the state.

The state can be global and owned. The difference between these two forms of
state can be illustrated with following statements:

* **Global state**: *nobody owns, everyone knows*

  Examples: asset name

* **Owned state**: *someone owns, nobody else knows*

  Examples: asset amounts, voting rights, NFT content

In our example we do not define a global state yet.

There are also three types of operations:
* **Genesis** - a unique operation which takes no input and sets up the contract;
* **State transition**, which changes some existing owned state ("inputs") and
  may add up to the global state;
* **State extensions**, which may be seen as a "decentralized additional
  geneses". State extensions do not have input state, but may produce new owned
  state.

The schema defines a rules of how the contract operates. Now let's put it into
a practice by creating a specific identity contract, and then doing a revocation:

```haskell
contract meSatoshiNakamoto implements DecentralizedIdentity
  -- this defines a genesis state and assigns it to a single-use-seal
  assign orig Identity := 
    (0xfac503c4641c3deda72a2d00bc9d6ff1094b15276c386efea403746a91436772, 1)
    -> PgpKey(0, 0x028730eeeec41802621d177507b086f390ae600ba3ca5e428b13913af4c2cd25b3)

transition iLostMyKey executes Revocation
  -- specifies the single-use-seal we close to match requirements
  -- on the valid operation execution conditions
  via meSatoshiNakamoto.orig
  
  -- here we use txid of the bitcoin transaction which will be
  -- created to hold the commitment to this state transition,
  -- called "single-use-seal witness". Since we can not know the
  -- txid upfront we use ~ to indicate the witness transaction id
  assign upd Identity := (~, 2) 
   -> PgpKey(0, 0x0219db0a4e0eb8cb833608c08d76b9b279ec44a851ab82cc6fd68a9b32624bfa8b)
   -- the above defines new state and assigns it to a single-use-seal
```

[strict types]: https://www.strict-types.org/
