+++
title = "RGB Node"
weight = 20
[extra]
anchor = "node"
bg-color = "red"
+++

RGB Node is a daemon/service which runs in the background, provides RPC API,
and allows you to always be online and accept incoming RGB transfers without 
any third-party server. You can install it on your server machine and then
interact with it locally or remotely by using `rgb-cli` command line utility.
You may also add your RGB Node address to RGB wallets that support it, which will
allow you to issue invoices using your own RGB node instance.

The detailed RGB node install instructions can be found in its 
[GitHub repo](https://github.com/RGB-WG/rgb-node#install).

**Please pay attention that the current version of RGB Node supports the previous
RGB protocol version, v0.9, and is not compatible with the latest protocol changes.
Until a new RGB Node v0.10 is released it is recommended to use the RGB command-line
tool as described in the previous section.**
