+++
weight = 20
title = "Programming RGB"
[extra]
header = false
+++

Contract business logic in RGB is defined through so-called *schema*; the 
specific RGB contract must implement some *schema*. One may think of a schema 
as a "class" definition in the OOP world; in that terms specific RGB contracts 
are "class instance" created by the schema constructor (*genesis operation*). 
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

## Writing simple contract

The best way to understand RGB is through example. Let's do a contract which 
will work as a digital identity.

We have to start with definition of a data type representing the key. This is 
not a part of the contract itself and may be imported into multiple contracts 
from a _data type library_.

```haskell
data PgpKey :: curve U8, key Bytes
```

AS you might see, the code looks similar to Haskel. New data types are declared
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

## Adding fungible state

Now, let's add some tokens to the contract, in form of "I owe you" obligations
provided by the decentralized identity:

```haskell
schema DecentralizedIdentity
   owned Identity :: PgpKey
   owned IOYIssue :: Zk64
   -- `Zk64` means 64-bit unsigned integer hidden with zero-knowledge
   owned IOYTokens :: Zk64

   global IOYTicker :: String
   global IOYName :: String

   genesis :: Identity, IOYTicker, IOYName

   op Revocation :: old Identity -> new Identity

   op Promise :: used IOYIssue -> given [IOYTokens]?, remaining IOYIssue?
      assert used == sum given + (remaining ?? 0)

   op Transfer :: spent {IOYTokens} -> received [IOYTokens]
      assert sum spent == sum received
```

Here we see multiple new things, let's try to explain them one by one.

First, it is the `global` keyword used to define asset ticker and name as two 
strings.

We already covered *owned state* composed of the rights holder and state atoms; 
but in some situations a contract needs a state which is "global" for the 
contract and is not "owned" by anyone (i.e. not assigned to a UTXO 
single-use-seal). The name of the asset (and ticker) is that sort of 
information: in the examples above it does not "belong" to a contract party but 
instead is a "property" of the contract itself. Another example could be the 
name of the identity holder which can't be changed and thus can't be assigned to
a UTXO single-use-seal. Such state is defined with the `global` keyword.

It is important to note, that while the global state is not "directly owned" by 
any party, it still may be updated through state transitions -- but only if 
schema enables that explicitly. In such cases the use of the global state (as 
compared to the owned state) is reasonable only when the user may opt-out from 
future state updates (i.e. do not define a new single-use-seals) but still needs
to keep the state. The asset name is again such an example, since even if the 
asset can be renamed, future renames after certain renaming may be prohibited 
(i.e. no new owned state can be created), but the asset name should stay. 
`global` allows exactly that.

Second, the use of braces, brackets and question mark around and next to the 
state name. Both braces and brackets mean that the operation works not with a 
single *state atom*, but instead an array (braces) or set (brackets) of *state 
atoms*. Set differs from an array in the fact that all elements of the set must
be unique, while in array they may repeat. Coming back to our operations, asset
transfer may spend a number of assets from different owners, but all of them 
must be unique (to prevent double-spend), i.e. form a set. However, when 
defining a receivers, multiple *state atoms* may be equal (i.e. you can send 
10 tokens to the same UTXO `N` times, such that the receiver will get `10*N` 
tokens in total, but through different "inputs", which may increase privacy).

Question mark is used to inform that the *state* may be absent, i.e. in case of
array or set it can be empty - or in case of a single item, it is optional.

At the very beginning we had mentioned so-called *state extensions*. State 
extension can be created by anybody without doing an on-chain commitment (i.e.
without closing any single-use-seals). In this it is similar to *genesis*. 
However, the state created by the extension is not "final" (compare to a 
non-mined transaction in the bitcoin mempool) until it gets included into a 
valid state transition (like transaction gets mined by being included into 
bitcoin valid block).

Probably the simplest way to understand state extensions is by example. Let's 
assume we'd like to do an asset which can be issued by anybody in the world 
through burning equivalent amount of bitcoins. Such operation can't be a state
transition, since we do not have a predefined set of single-use-seals to define
a "rights of issue" for an open and unknown set of participants. We can't also
do "multiple geneses", since each genesis will define a new RGB contract and 
the assets under different RGB contracts are not fungible. State extensions 
were created exactly to address this issue. A contract using them may look in
the following way:

```haskell
-- We need some basic bitcoin-related types
-- We can do `import BP.{Utxo, RedeemScript, ...}`, but here we'd like to use
-- this opportunity to showcase how type definitions work
data Utxo :: Txid, vout U16
data Txid :: [Byte^32]
data RedeemScript :: Bytes
data CompressedPk :: [Byte^33]
data XonlyPk :: [Byte^32]
data OutputDescriptor :: Wpkh | Tr
data Wpkh :: CompressedPk
data Tr :: XOnlyPk

data ProofOfBurn ::
  burned Utxo,
  descriptor OutputDescriptor
  -- output descriptor is used to prove that the bitcoins can't be spent.
  -- in the validation logic below we define that PoB output must be either
  -- SegWit v0 WPKH or taproot output with a keypath-spending; in both cases
  -- the key given as a prove must be tweaked with its own hash, which will
  -- construct provably-unspendable key

schema RgbWrapBtc
  global Ticker :: String
  global Name :: String

  owned Value :: Zk64 -- the assets will be ownable and transferrable as any
                      -- other fungible asset

  valency Burn -- the valency declares a possibility of the public 
               -- participation in the contract logic.
               -- If the contract may have different forms of public 
               -- participation, each of them must be declared first.

  genesis :: Ticker, Name, -- in genesis we can't issue anything, we just
                           -- declare specific name and ticker for our wrapped
                           -- bitcoin
             Burn -- we also say that the genesis allows state extensions to
                  -- be directly linked to the contract

  -- this is the state extension procedure: a "public" method of the contract
  -- which takes information about the proof of burn and allows to issue the
  -- wrapped bitcoins assigned to a single-use-seal(s), which are lately can
  -- be transferred with a usual state transition.
  public Issue :: Burn, proof ProofOfBurn -> issued [Value]
    -- here we verify that the UTXO input constructed with the key given in
    -- the proof according to the provably-unspent procedure exactly matches
    -- output existing onchain
    let spk' = match proof.descriptor
      Wpkh(pk) =>
        pk |>
        Hash.sha256 |>              -- h = Hash.sha256 pk
        SECP.gen |>                 -- tweak = SECP.gen h
        SECP.add pk |>              -- pk' = SECP.add pk, tweak
        Hash.hash160 |>             -- h' = Hash.hash160 pk''
        BP.wpkhScriptPubkey         -- BP.wpkhScriptPubkey h'
      Tr(xpk) =>
        xpk |>
        BIP340.selfTweak |>         -- xpk' = BIP340.selfTweak xpk
        BP.trScriptPubkey           -- BP.trScriptPubkey xpk'

    let spk = assert BP.readScriptPubkey proof.burned
    assert spk == spk'
    assert BP.notSpent proof.burned
    assert BP.value proof.burned == sum issued

  op Transfer :: spent {Value} -> received [Value]
    assert sum spent == sum received
```

## Contract intefaces

But how the wallet makes the sense of the contract? RGB contract can do a lot of things, and if schema developers would need to do a separate wallet for each schema the entrance threshold would be too high. To avoid such situation a concept of *contract interface* was created. A *contract interface* is a standard way communicating with RGB Node asking it for a semantically-meaningful state and creating operations. Such concept of interface is similar to the concept of ERC standards and ABI files in Ethereum world; the most common interfaces are called "RGBxx" and are defined as a separate LNP/BP standards.

Here, instead of using existing interface standards we will create an interface for a generic fungible token from scratch to explain the way they work:

```haskell
interface FungibleToken:
   global Ticker -> String -- this is similar to schema definition; in fact
                           -- it is a requirement that the schema must provide
                           -- a global state of the String type and link it to
                           -- the "Ticker" name
   global Name -> String

   -- amount of the issued asset must be a owned state which should be 
   -- accessible to all contract participats, hence it is `public`
   owned public Inflation :: Zk64
   owned Asset :: Zk64

   op Issue :: Inflation -> [Asset]?, Inflation? -- and operations
   op Transfer :: {Asset} -> [Asset]

-- Specific schema state may use different naming, for instance because a 
-- schema can define multiple assets with different names; in that case we 
-- will have multiple interface implementations referencing different state.
implement FungibleToken for DecentralizedIdentity
   global Ticker := IOYTicker -- this creates a _binding_ of the state defined
                              -- in the schema (*IOYTicker* in this case) to
                              -- the interface
   global Name := IOYName
   owned Inflation := IOYIssue
   owned Asset := IOYTokens
   op Issue := Promise
   op Transfer -- here we skip `:=` part since the interface operation name
               -- matches the name used in the schema. In such cases we can 
               -- also skip the declaration at whole
```

RGB smart contract can implement multiple interfaces. In our case it would be 
desirable to expose the identity information as well:

```haskell
interface PgpIdentity
  owned Identity :: PgpKey
  exec Revocation :: old Identity -> new Identity

implement PgpIdentity for DecentralizedIdentity
  -- we do not need to put anything here since schema state and operation
  -- names matches interface requirements and the compiler is able to guess
  -- the bindings
```

[AluVM]: https://www.aluvm.org/
[strict types]: https://www.strict-types.org/