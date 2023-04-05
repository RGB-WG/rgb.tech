+++
weight = 30
title = "Contract interfaces"
[extra]
bg-color = "purple"
+++

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
