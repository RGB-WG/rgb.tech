+++
title = "RGB release v0.7"
date = 2022-06-13
+++

We are happy to announce v0.7 release of RGB v0.7, which provides a milestone
of the recent RGB developments which include:
* Full taproot-based OP_RETURN commitments (**tapret**)
* New multi-protocol commitments (**MPCs**) codified as LNPBP-4 standard. They
  are based on merkle trees and provide support for using RGB alongside other 
  client-side-validation protocols.
* State transition **bundling** for multi-participant transactions (required for 
  LN and payjoins/coinjoins).
* **Data attachments** (binary data for NFTs etc) and support for arbitrary rich
  data in contract state with a new schema for representing hierarchies of such
  data structures.

Alongside with RGB Core libraries it we have released RGB Node v0.7, which is a
developer preview to test all those new RGB features. The RGB Node API is 
subject to change with the new v0.8 version (which will include many 
wallet/PSBT-related, data storage and networking) changes, so it is advised not
to invest a lot of effort into integrating this version and rather wait for v0.8
"in two weeks" :).
