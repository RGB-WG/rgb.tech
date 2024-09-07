+++
title = "State transfers"
weight = 20
[extra]
bg-color = "purple"
anchor = "transfer"
+++

In most of the cases interaction between contract participants happens through
**invoicing**: if *Alice* wants *Bob* to do something with the contract state he
owns (for instance, send some of the tokens to *Alice*), she constructs an
invoice which contains instructions for Bob on the action he is asked to perform.

Hereinafter we will call Alice, who **issues an invoice**, a *beneficiary*, and
Bob, who **executes** the invoice, an *actor*. We say *invoice executing*
instead of *paying the invoice*, since invoice may not contain a payment and
carry some other form of action (key revocation, casting a vote etc).


### Creating an invoice

In RGB, invoicing is much more general than just an invoice for a certain amount
of tokens. In fact, you can pack any request for any contract operation, under
any possible schema and interface in form of invoice: revoking identity, voting
in DAO, performing secondary asset issuance, creating NFT engraving and so on,
so on.

To construct an invoice one have to run the following command:

```sh
$ rgb invoice $CONTRACT -i $INTERFACE -a $ACTION $STATE $SEAL
```

Here:
- **`$CONTRACT`** should be a contract id under which the interaction happens
- **`$INTERFACE`** is human-readable interface name which should be used for
  interacting the contract by both parties, beneficiary and actor. It may be
  skipped, if the contract schema implements just a single interface.
- **`$ACTION`** is the name of the operation which should be performed by the
  actor. If it is skipped, it defaults to an operation which declared in the
  used interface as "default".
- **`$STATE`** provides an information about the state, which should be
  transferred to the beneficiary single-use-seal defined in the `$SEAL`
  parameter. The state mast match state type for the given action, for instance,
  for a transfer of a fungible token it must be an integer specifying amount
  of smallest token denominations to be transferred.
- **`$SEAL`** is a definition of a single-use-seal owned by beneficiary; it can
  be either a UTXO on beneficiary's wallet, or an address. If an address is used
  then the actor will use it to construct an UTXO in his witness transaction,
  and this UTXO will be seen and in control of the beneficiary.

An example of the command using RGB20 interface to request a token transfer:

```sh
alice$ CONTRACT='D4RN7r4$-ZNt43c$-ymINZ1r-M$bJPPf-SWp9193-OLIdtv0'

alice$ MY_UTXO=4960acc21c175c551af84114541eace09c14d3a1bb184809f7b80916f57f9ef8:1

alice$ rgb invoice $CONTRACT -i RGB20 100 $MY_UTXO
```

The result of this commands would be an invoice printed to `STDOUT`:
```
rgb:D4RN7r4$-ZNt43c$-ymINZ1r-M$bJPPf-SWp9193-OLIdtv0
/RGB20/BF+bc:utxob:zlVS28Rb-amM5lih-ONXGACC-IUWD0Y$-0JXcnWZ-MQn8VEI-B39!F
```

To learn more about invoices please refer to [RGB FAQ Website](https://rgbfaq.com/glossary/user/invoice)


### Performing transfer

To perform a state transfer *Bob* has to receive an invoice from *Alice*.
We assume that *Bob* has some command-line wallet tool, which is able to
construct, sign and publish normal bitcoin transactions (we will represent
this tool with a `wallet` command).

At the first stage, Bob constructs PSBT file, which must spend outputs
containing sufficient amount of RGB assets, plus an output to store a change.
We assume that Bob saves PSBT to `tx.psbt` file.

Now, Bob can pay Alice's invoice (which he saved to `$INVOICE` variable) with
the following command:

```sh
bob$ rgb transfer tx.psbt $INVOICE consignment.rgb
```

The result transfer consignment will be stored to `consignment.rgb` file in a
binary form. Bob should send this file to Alice by some third-party means - this
can be an e-mail, some file server – or one of existing RGB-related protocols,
like RGB-RPC, Storm (no Lightning network) or something else.

Upon receiving the consignment Alice verifies and accepts it to her local
**RGB stash** (which keeps information about all Alice contracts and owned
state) with the following command:

```sh
alice$ rgb accept consignment.rgb
```

If the consignment was invalid, the command will fail and list all problems the
consignment had. Otherwise, the command should succeed with a single warning –
that the terminal witness transaction is not yet mined. This is due to the fact
that Bob hasn't yet published his transaction yet, since he is waiting on Alice
approval that she is happy with the transfer.

Upon adding transfer consignment to the stash command will return a signature
over the consignment, which Alice can send back to Bob as a form of payslip.

Bob can now check Alice's signature, sign and publish his transaction:

```sh
bob$ rgb check <sig> && wallet sign --publish tx.psbt
```

Alice's RGB wallet will see the witness transaction mined, after which the
new state will appear in her user interface.
