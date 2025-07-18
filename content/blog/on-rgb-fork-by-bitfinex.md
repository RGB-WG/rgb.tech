+++
title = "Security notice on RGB fork claimed to be 'RGB v0.11.1'"
date = 2025-07-18
authors = ["Maxim Orlovsky"]
description = "A large-scale change to the outdated RGB consensus codebase is 'released to production' without passing peer-review"
+++

It has come to our attention that the recently announced "release" of "RGB protocol version 0.11.1"
runs a fork of RGB v0.11 from unreleased last year codebase, with heavily modifications to the consensus, affecting its security.
These modifications were proposed to the main RGB repositories but didn't meet our security requirements
and were of poor code quality (one may refer to the following [link](PR) for example)

Specifically, its unit test coverage is nearly absent
(compared to the full test coverage of v0.12 [released by us recently][v0.12], see the image below);
it lacks a lot of functionality specific to zk support; it has removed smart contracting capabilities;
has heavy performance and scalability issues etc.

![Test coverage](/blog/v0-12-tests.png)

Thus, while as a free (libre) software project we do not oppose experiments and fork,
we can't prevent possible malicious fork code from entering these forks,
nor can't prevent users from using the forked code.
Thus, we do not advise any production use of version 0.11.1;
the only version recommended by us for the production is v0.12 from https://github.com/RGB-WG organization.

Companies may check whether the forked and changed consensus is used in their dependency trees
by running cargo tree -i rgb-consensus, which will show whether the forked and modified version
of official rgb-core consensus crate is being used.
If this is the case, we recommend to update the software or do not use in production.

We also advise users NOT to use products built with version v0.11.1 and
modified consensus for any assets or purposes with any real value.

We also recommend to change the name of the forked version from RGB Protocol, to avoid user confusion and misguidance.

In any case, if a loss of assets would happen, neither the real RGB protocol, 
nor LNP/BP Standards Association or RGB Consortium can be blamed for it
(with claims alike "it is based on your code"),
giving the facts that that code was never released and recommended by us for the production,
and dramatic scale of consensus modification and code changes made in the fork.

We will also not support the forked version of RGB as we do not have any resources to perform its audit or analysis;
and the fork maintainers are not interested in our critics, rejecting a normal route of PR peer reviews.

[PR]:  https://github.com/RGB-WG/rgb-core/pull/288#pullrequestreview-2876658349
[v0.12]: https://rgb.tech/blog/release-v0-12-consensus/
