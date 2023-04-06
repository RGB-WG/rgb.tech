+++
title = "Issuing contract"
weight = 30
[extra]
bg-color = "red"
anchor = "issue"
+++

To issue a contract you need to write a contract declaration file. Contract
declaration can be done in multiple languages, including Rust, YAML, TOML, JSON
and future [Contractum](https://www.contractum.org). Here we will stick with
a YAML format as most human-readable:

```yaml
interface: RGB20

globals:
  Nominal:
    ticker: TEST
    name: Test asset
    details: ~
    precision: 8
  ContractText: >
    SUBJECT TO, AND WITHOUT IN ANY WAY LIMITING, THE REPRESENTATIONS AND WARRANTIES OF ANY SELLER 
    EXPRESSLY SET FORTH IN THIS AGREEMENT OR ANY OTHER EXPRESS OBLIGATION OF SELLERS PURSUANT TO THE
    TERMS HEREOF, AND ACKNOWLEDGING THE PRIOR USE OF THE PROPERTY AND PURCHASER’S OPPORTUNITY 
    TO INSPECT THE PROPERTY, PURCHASER AGREES TO PURCHASE THE PROPERTY “AS IS”, “WHERE IS”, 
    WITH ALL FAULTS AND CONDITIONS THEREON. ANY WRITTEN OR ORAL INFORMATION, REPORTS, STATEMENTS, 
    DOCUMENTS OR RECORDS CONCERNING THE PROPERTY PROVIDED OR MADE AVAILABLE TO PURCHASER, ITS AGENTS
    OR CONSTITUENTS BY ANY SELLER, ANY SELLER’S AGENTS, EMPLOYEES OR THIRD PARTIES REPRESENTING OR
    PURPORTING TO REPRESENT ANY SELLER, SHALL NOT BE REPRESENTATIONS OR WARRANTIES, UNLESS
    SPECIFICALLY SET FORTH HEREIN. IN PURCHASING THE PROPERTY OR TAKING OTHER ACTION HEREUNDER,
    PURCHASER HAS NOT AND SHALL NOT RELY ON ANY SUCH DISCLOSURES, BUT RATHER, PURCHASER SHALL RELY
    ONLY ON PURCHASER’S OWN INSPECTION OF THE PROPERTY AND THE REPRESENTATIONS AND WARRANTIES 
    HEREIN. PURCHASER ACKNOWLEDGES THAT THE PURCHASE PRICE REFLECTS AND TAKES INTO ACCOUNT THAT THE
    PROPERTY IS BEING SOLD “AS IS”.

assignments:
  Assets:
    seal: tapret1st:01d46e52c4bdb51931a0eae83e958c78bdef9cac2057b36d55370410edafdd42:0
    amount: 100000000000000
```

As you can see, contract definition consists of:
* specification of the interface used for constructing the contract and parsing
  provided state
* global state – in this case specifying token nominal information and giving
  legal contract test
* assignments, which assign owned state (issued asset amount) to a single-use-seals,
  which are bitcoin UTXO prefixed with a type of seals (tapret 1st output).

You compile the contract with the command

```sh
$ rgb issue 4h3xkAmiRdQs2HwQmnFVqoj5DG3NhGkJRpNZXjNrJT2N RGB20 examples/rgb20-demo.yaml
```

Here, we use a default schema for RGB20 assets, which is provided with RGB 
itself, but one may develop a custom schema as described in 
[programming RGB](/program) part of this website. We also specify an interface
(RGB20) and provide a YAML file to which we have saved our contract.

The issued contract was added to the stash. You may see it by running

```sh
$ rgb info
```

```
Schemata:
---------
marina-karl-basic-4h3xkAmiRdQs2HwQmnFVqoj5DG3NhGkJRpNZXjNrJT2N: RGB20 

Interfaces:
---------
RGB20 complex-field-union-DbwzvSu4BZU81jEpE9FVZ3xjcyuTKWWy2gmdnaxtACrS

Contracts:
---------
rgb:BasilYellowGrand0BgHH3yYaCUvaw5Tc8KfrVB1aWN4vBvCXCuMfK98N3o6Z
```

To distribute contract to other users you have to export it in a form of
**contract consignment**. This consignment may be a binary file - or 
ASCII/Base64 armored text, such that it can be sent via messengers or by an
e-mail (or even put into a dynamic QR code).

```sh
$ rgb export -a BasilYellowGrand0BgHH3yYaCUvaw5Tc8KfrVB1aWN4vBvCXCuMfK98N3o6Z
```

We provide an `-a` option before the contract id to use ASCII armoring. If we
do not provide a file name as a last argument, the contract is exported to
the standard output of the terminal:

```
----- BEGIN RGB CONTRACT -----
Id: BgHH3yYaCUvaw5Tc8KfrVB1aWN4vBvCXCuMfK98N3o6Z
Checksum: basil-yellow-grand

AQAAAAACAAAqtIS1DeMje22cf/4RnKx+xPCNaqxfFqvWGhpQNPOrpAEAAQDw06ivRjpjzvxKOiEt
SfSRNwdqeH5viPmGMD0IY93KMAEAAQAAAQgAxr3uYkgVMfD14HWM2mhgU8BQxPxK0SGaXFuOW/3B
trECAAABAAEAAQABAAEAAQAAAQD//wAAAQAAxr3uYkgVMfD14HWM2mhgU8BQxPxK0SGaXFuOW/3B
trEAAQAAAQD//wEAAAEA//8ADwAAIoM2c5ybCMor86xCND2RseewiCxZlc49r51r9H/TA0UBC1JH
QkNvbnRyYWN0DENvbnRyYWN0TmFtZQUB5YDcz29OYj8l60N4tNmXBXNso2tmZhosAPikieRyt6sq
tIS1DeMje22cf/4RnKx+xPCNaqxfFqvWGhpQNPOrpAELUkdCQ29udHJhY3QHTm9taW5hbAYEBnRp
Y2tlcpLnXKh6DUXvMZ3Zc5AtjaLSaC5MfiPfR5v+e+hCmDUGBG5hbWUigzZznJsIyivzrEI0PZGx
57CILFmVzj2vnWv0f9MDRQdkZXRhaWxzNlYTwDFDkvlzaxCwNmRv+WzqeIWhnBnyDfvUtmQDTkoJ
cHJlY2lzaW9uykNH38AFMQoa5Vkrfdbz6TZF5SMTBfvpku8zziza4x42VhPAMUOS+XNrELA2ZG/5
bOp4haGcGfIN+9S2ZANOSgAEAgAEbm9uZca97mJIFTHw9eB1jNpoYFPAUMT8StEhmlxbjlv9wbax
AQRzb21l0KC6y1ZHRS3FNas6lTWmmaVfqG/yORKXGb8uUARjlBQ+VY/6Dww4withvk0RC4k2lNUc
U0xwbzSK8xMlHskDNgAIYc2QPfO0+7TYKNNcU/l24/gdyeEYILKqdmYeTSiiVRAAAAAAAAAAAP//
AAAAAAAARG+7cbgnA35t9/gdg4KINl4VK6TcQG1qQ5INyYpsbyUACGHNkD3ztPu02CjTXFP5duP4
HcnhGCCyqnZmHk0oolUQKAAAAAAAAAD/AAAAAAAAAFV37iE4le7TpKToItt89gePS14g13JwujwQ
v4lzxs1LAANgA18zMiADXzMzIQNfMzQiA18zNSMDXzM2JANfMzclA18zOCYDXzM5JwNfNDAoA180
MSkDXzQyKgNfNDMrA180NCwDXzQ1LQNfNDYuA180Ny8DXzQ4MANfNDkxA181MDIDXzUxMwNfNTI0
A181MzUDXzU0NgNfNTU3A181NjgDXzU3OQNfNTg6A181OTsDXzYwPANfNjE9A182Mj4DXzYzPwNf
NjRAA182NUEDXzY2QgNfNjdDA182OEQDXzY5RQNfNzBGA183MUcDXzcySANfNzNJA183NEoDXzc1
SwNfNzZMA183N00DXzc4TgNfNzlPA184MFADXzgxUQNfODJSA184M1MDXzg0VANfODVVA184NlYD
Xzg3VwNfODhYA184OVkDXzkwWgNfOTFbA185MlwDXzkzXQNfOTReA185NV8DXzk2YANfOTdhA185
OGIDXzk5YwRfMTAwZARfMTAxZQRfMTAyZgRfMTAzZwRfMTA0aARfMTA1aQRfMTA2agRfMTA3awRf
MTA4bARfMTA5bQRfMTEwbgRfMTExbwRfMTEycARfMTEzcQRfMTE0cgRfMTE1cwRfMTE2dARfMTE3
dQRfMTE4dgRfMTE5dwRfMTIweARfMTIxeQRfMTIyegRfMTIzewRfMTI0fARfMTI1fQRfMTI2fgRf
MTI3f1zQw8DbZ34iXVDAzVWjj9KnnTop1iXjT3jf1hZsPLgGAAhVd+4hOJXu06Sk6CLbfPYHj0te
INdycLo8EL+Jc8bNSwEAAAAAAAAACAAAAAAAAABhzZA987T7tNgo01xT+Xbj+B3J4Rggsqp2Zh5N
KKJVEAABb8aWNwahCDQgGejxadp+9IqPQ8ioGIMkRvPuF6i1ObwBC1JHQkNvbnRyYWN0D0NvbnRy
YWN0RGV0YWlscwUBRG+7cbgnA35t9/gdg4KINl4VK6TcQG1qQ5INyYpsbyWS51yoeg1F7zGd2XOQ
LY2i0mguTH4j30eb/nvoQpg1BgELUkdCQ29udHJhY3QGVGlja2VyBQFc0MPA22d+Il1QwM1Vo4/S
p506KdYl409439YWbDy4Bsa97mJIFTHw9eB1jNpoYFPAUMT8StEhmlxbjlv9wbaxAAAAykNH38AF
MQoa5Vkrfdbz6TZF5SMTBfvpku8zziza4x4BC1JHQkNvbnRyYWN0CVByZWNpc2lvbgMRC2luZGl2
aXNpYmxlAARkZWNpAQVjZW50aQIFbWlsbGkDBW1pY3JvBglkZWNpTWljcm8HCmNlbnRpTWljcm8I
BG5hbm8JCGRlY2lOYW5vCgljZW50aU5hbm8LBHBpY28MCGRlY2lQaWNvDQljZW50aVBpY28OBWZl
bXRvDwlkZWNpRmVtdG8QCmNlbnRpRmVtdG8RBGF0dG8S0KC6y1ZHRS3FNas6lTWmmaVfqG/yORKX
Gb8uUARjlBQABQFvxpY3BqEINCAZ6PFp2n70io9DyKgYgyRG8+4XqLU5vOWA3M9vTmI/JetDeLTZ
lwVzbKNrZmYaLAD4pInkcrerAAhhzZA987T7tNgo01xT+Xbj+B3J4Rggsqp2Zh5NKKJVEAUAAAAA
AAAAKAAAAAAAAADw06ivRjpjzvxKOiEtSfSRNwdqeH5viPmGMD0IY93KMAELUkdCQ29udHJhY3QM
Q29udHJhY3RUZXh0BQE+VY/6Dww4withvk0RC4k2lNUcU0xwbzSK8xMlHskDNgABDieh3fpTsCEB
UDTfigOMqoQNnWUf0itqpviSlmrfVn4MAANSR0IDANAAAAAAAAEABAAADieh3fpTsCEBUDTfigOM
qoQNnWUf0itqpviSlmrfVn4AAAGWwcTJa9GZ/1ploGCamFxyZysjojs4MmiJQPGX4wL3AQVSR0Iy
MAIMQ29udHJhY3RUZXh0AfDTqK9GOmPO/Eo6IS1J9JE3B2p4fm+I+YYwPQhj3cowAQdOb21pbmFs
ASq0hLUN4yN7bZx//hGcrH7E8I1qrF8Wq9YaGlA086ukAQEGQXNzZXRzAAIAAAIMQ29udHJhY3RU
ZXh0AQABAAdOb21pbmFsAQABAAEGQXNzZXRzAQD//wABCFRyYW5zZmVyAAABBkFzc2V0cwEA//8B
BkFzc2V0cwEA//8AAQZBc3NldHMAAQhUcmFuc2ZlcjbVpCrKcaQDIWfTc4Sl6JvDFJkaRvimk4G0
iH28yGhPlsHEyWvRmf9aZaBgmphccmcrI6I7ODJoiUDxl+MC9wECAAAHTm9taW5hbAEADENvbnRy
YWN0VGV4dAEAAAZBc3NldHMAAQAACFRyYW5zZmVyAAAAADbVpCrKcaQDIWfTc4Sl6JvDFJkaRvim
k4G0iH28yGhPgwAAAgAAAQASAARURVNUClRlc3QgYXNzZXQACAEAAQACAAAAAQAAAQEAAwFiNVSs
Hc0VSWwQWicELEOJIfKoKHNXm+iOdNfvVZo9kQAAAAA9qe0Y5e3D8QgAQHoQ81oAAHvqdgO4GMt1
N/GyBwEZNIJHD4eLef1tItTO/SxpB6mwAAAAAAAAAAAAAAAAAAA=

----- END RGB CONTRACT -----
```
