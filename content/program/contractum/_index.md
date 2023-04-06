+++
title = "Programming with Contractum"
sort_by = "weight"
[extra]
multicolumn = true
+++

[**Contractum**](https://www.contractum.org) is a functional & declarative 
language for writing smart contracts which is a work-in-progress. The first 
compiler version is planned for a release by the end of 2023; before that you 
may use this section as a guideline and preview of how the development of RGB 
smart contracts with contractum may look like in a future.

Contractum is a domain-specific language created with more general functional 
language named **ParselTongue**. It uses 
[**Strict Types**](https://www.strict-types.org) formalism for defining data 
types and is compiled into RGB *schema*, *interfaces*, *interface 
implementations*, *genesis*, *contract operations* and 
[**AluVM**](https://www.aluvm.org) executable code. To learn more about RGB 
schema, interfaces and other related concepts please refer to 
[RGB Components](/program/#components) section and [RGB FAQ](https://www.rgbfaq.com)
website.

We will demonstrate how RGB with Contractum works in three main steps:

- <a href="#basics" class="button button-secondary">Writing simple contract</a>
- <a href="#fungibles" class="button button-secondary">Adding fungible state</a>
- <a href="#interfaces" class="button button-secondary">Implementing interfaces</a>
