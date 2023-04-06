+++
weight = 20
title = "Adding fungible state"
[extra]
bg-color = "green"
+++

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
single *state atom*, but instead an array (brackets) or set (braces) of *state
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
    -- output existing on-chain
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
