---
layout: default
title: "Documentation"
---

![](https://github.com/mediachain/mediachain/blob/master/mediachain_logo_small.png?raw=true)

Welcome to the documentation for the [Mediachain project][mediachain-io], an open, universal
media library.

Checkout the [Installation Guide][install] to get the mediachain client and daemon software
installed on your machine, or [deploy a mediachain node to the cloud][cloud-deploy] with one click.

Once the node is installed, the [Getting Started Guide][getting-started] will help you get your node
set up and ready to use. Using mediachain to [query peers][basic-ops-query], [publish metadata statements][basic-ops-publish],
[merge remote data][basic-ops-merge] and more are covered in the [Basic Operations Guide][basic-ops].

## What is Mediachain?
Mediachain is a singular data fabric for open-first media applications.

It serves the role of a traditional database, but is a decentralized, global data layer for powering serverless applications.

It is a single port of entry for applications and users to publish, discover, and collaborate on media metadata.

With Mediachain, a group of museums can collaborate on [cultural heritage data](https://blog.mediachain.io/bringing-cultural-metadata-to-life-12cc118b2298) in a shared system, a cooperative of openly-licensed image sharing platforms can publish [attribution information](https://blog.mediachain.io/introducing-mediachain-attribution-engine-2dc1ea6aa31f) to a community-maintained ledger, a consortium of music industry organizations can share [rights data](https://blog.mediachain.io/what-a-blockchain-for-music-really-means-e2f8dc66d57d#.g5kmvz3jh) without ceding control to a third party, and a developer can build a decentralized blogging platform without needing to run a centralized database.

Mediachain complements emerging technologies in the decentralized application protocol stack, where Ethereum is the logic layer, IPFS is the file system, and Mediachain is the database.

Unlike traditional DHT and blockchain-based systems, Mediachain scales efficiently when publishing billions of small records, and can accommodate use-cases like archiving high cardinality datasets or building internet-scale decentralized media applications.

In short, Mediachain lets developers build dynamic, social, and collaborative applications like the ones that defined Web 2.0, but in a completely decentralized way. With Mediachain, participants are in control of their identity and their data, and value is exchanged without intermediaries.

## How it works

![](https://github.com/mediachain/mediachain-docs/raw/master/images/mc-stack.png)

[mediachain-io]: https://www.mediachain.io
[install]: {{site.baseurl}}{% link _docs/setup/install.md %}
[cloud-deploy]: {{site.baseurl}}{% link _docs/setup/cloud-deploy.md %}
[getting-started]: {{site.baseurl}}{% link _docs/setup/getting-started.md %}
[basic-ops]: {{site.baseurl}}{% link _docs/usage/basic-operations.md %}
[basic-ops-query]: {{site.baseurl}}{% link _docs/usage/basic-operations.md %}#query
[basic-ops-publish]: {{site.baseurl}}{% link _docs/usage/basic-operations.md %}#publishing-statements
[basic-ops-merge]: {{site.baseurl}}{% link _docs/usage/basic-operations.md %}#merge
