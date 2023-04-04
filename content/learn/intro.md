+++
weight = 10
title = "RGB Overview"
[extra]
bg-color = "green"
+++

RGB is a full-fledged smart contract system which operates on top of Bitcoin
and Lightning network. If you think about blockchain as a settlement layer and
Lightning as a scaling layers, *RGB is the programmability layer*, finally 
adding to the first and most censorship-resistant cryptographic-based store of
value properties which were absent and desired for many years.

RGB achieves this though a separation of abstraction layers: the bitcoin
blockchain (layer 1) becomes a cryptographic commitment layer, which holds
*no data* but a compact commitments to the smart contract state, which is
now kept off-chain. These commitments do not occupy block space, since they
are kept inside taproot tree and not directly visible onchain.

Such approach also allows RGB to operate on top of Lightning network, since
it can work with any bitcoin transaction, including transactions incorporated
into the channel. RGB scripts and state fully exists in the client-side world,
not seen from outside (hence privacy) - and not required to be validated by
anyone else than the contract participants (hence scalability).

RGB makes bitcoin to pass beyond limitations of [CIA triad] via achieving 
different properties on different layers:
* **Blockchain** provides integrity & availability;
* **Lightning network** – integrity & confidentiality;
* **RGB** – confidentiality & availability.

[CIA triad]: https://www.coursera.org/articles/cia-triad
