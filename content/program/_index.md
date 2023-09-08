+++
title = "Program RGB"
sort_by = "weight"
+++

RGB is a full-fledged smart contract system which operates on top of Bitcoin
and Lightning network. If you think about blockchain as a settlement layer and
Lightning as a scaling layer, *RGB is the programmability layer*, finally
adding to the first and most censorship-resistant cryptographic-based store of
value properties which were absent and desired for many years.

RGB achieves this through a separation of abstraction layers: the bitcoin
blockchain (layer 1) becomes a cryptographic commitment layer, which holds
*no data* but compact commitments to the smart contract state, which is
kept off-chain. These commitments do not occupy block space, since they
are kept inside taproot tree and not directly visible on-chain.

Such approach also allows RGB to operate on top of the Lightning network, since
it can work with any bitcoin transaction, including transactions incorporated
into a channel. RGB scripts and state fully exist in the client-side world, are
not seen from the outside (hence privacy) - and are not required to be validated by
anyone other than the contract participants (hence scalability).

RGB allows Bitcoin to pass beyond the limitations of the [CIA triad] (confidentiality,
integrity, and availability) by achieving different properties on different layers:
* **Blockchain** provides integrity & availability;
* **Lightning network** – integrity & confidentiality;
* **RGB** – confidentiality & availability.

[CIA triad]: https://www.coursera.org/articles/cia-triad
