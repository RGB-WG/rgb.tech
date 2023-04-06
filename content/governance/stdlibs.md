+++
title = "Standard libraries"
weight = 20
[extra]
bg-color = "purple"
+++

Standard libraries is the lowest level for integration of RGB into other 
software. Changes in standard libraries do not affect the safety of RGB assets
and security of contracts, however may break software compatibility. Thus,
this layer is maintained by [LNP/BP Standards Association][LNP/BP], which  
ensures gradual changes and stability of APIs, alongside their improvements.

Standard libraries include:
* **RGB standard library**
* **RGB data type library**
* **RGB AluVM libraries**

Since the code in standard libraries may be used for a deep-level hacks,
it is recommended that all users of RGB interact with the system via standard
libraries reviewd by [RGB working group](RGB-WG) of LNP/BP Standards 
Association.

[LNP-BP]: https://lnp-bp.org
[RGB-WG]: https://github.com/RGB-WG
