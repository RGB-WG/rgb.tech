+++
title = "RGB as a Computing Platform"
date = 2023-05-21
authors = ["Maxim Orlovsky"]
description = "Like each of the other computing platforms (OS, Web, embedded, cloud, blockchain-based, VM-based) it has its own distinctive features"
+++

RGB is a computing platform.

Like each of the other computing platforms (OS, Web, embedded, cloud, blockchain-based, VM-based) it has its own
distinctive features.

Unlike blockchain-based computing platforms, it has access to ephemeral state data, which may be a part of the Lightning
channel state, or data provided by a decentralized data network. This is possible since in client-side validation,
unlike in blockchain, a single contract may have an invalid state and this doesn’t affect the state of the platform as a
whole. For instance, in Ethereum, if an invalid transaction under some contract is included in the blockchain, the whole
blockchain becomes invalid (and a different tip is selected). In RGB no global consensus on the validity of all
contracts and transactions is required.

RGB isolates each of the programs (“smart contracts”) in its sandbox environment, which provides much better scalability
and security than blockchain-based platforms.

Unlike device-based and Web platforms, RGB doesn’t provide random memory access, I/O, or UI, which makes RGB well-suited
for embedded devices and environments.

One of the distinctive features of the platform is the use of the functional registry-based virtual machine (#AluVM) and
functional type system.

RGB is the first computing platform utilizing PRISM computing model, which is closer to cellular automation computing
than instruction-based or neural networks. PRISM stands for “partially replicated state machines”, which at their core
represent a highly-parallel multi-agent system made with a functional approach.

Today, RGB (together with AluVM) can be run on x86, AMD64, Aarch64, microcontrollers, and WASM instruction set
architectures, i.e. it is a ubiquitous platform (desktop, mobile, server, embedded, Web).