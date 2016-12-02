---
title: Getting Started
layout: page
category: setup
order: 3
---

* TOC
{:toc #table_of_contents}

Welcome to Mediachain! This guide will help you get started with a newly-installed mediachain node.
For help with installation, please check out the [Installation Guide]({{ site.baseurl }}{% link _docs/install.md %}), then come back
here.

This guide will assume that you've installed `mcnode` and `mcclient` on your local computer, and
you want to get your node ready to use.  If you've [deployed your node
to the cloud]({{ site.baseurl }}{% link _docs/cloud-deploy.md%}), everything will work just the same, but you'll need to
[set up an SSH tunnel to control your node]({{ site.baseurl }}{% link _docs/ssh-tunnel.md%}).

## Starting a new node
Let's start our node!

You can run your local mediachain node by invoking `mcnode` without arguments:

```
$ mcnode
2016/10/20 19:29:24 Generating new node identity
2016/10/20 19:29:25 Saving key to /home/vyzo/.mediachain/mcnode/identity.node
2016/10/20 19:29:25 Node ID: QmeBkfxcaBfA9pvzivRwhF2PM7sXpp4HHQbp7jfTkRCWEa
2016/10/20 19:29:25 Generating new publisher identity
2016/10/20 19:29:25 Saving key to /home/vyzo/.mediachain/mcnode/identity.publisher
2016/10/20 19:29:25 Publisher ID: 4XTTMADSKQUN3jkeZngbtuE35w9y5YnDTicVTeeji7N2Npkey
2016/10/20 19:29:25 Node is offline
2016/10/20 19:29:25 Serving client interface at 127.0.0.1:9002
```

The first time you run `mcnode`, it will generate a pair of persistent identities and
initialize the local store. By default, `mcnode` uses `~/.mediachain/mcnode` as its
root directory, but you can change this using the `-d path/to/mcnode/home` command line option.

## Bringing the node online
By default, the node starts disconnected from the p2p network and provides a client control
api in localhost.
You can control and use your node through the api using curl, but it is recommended
to [install Aleph]({{ site.baseurl }}{% link _docs/install.md %}#installing-aleph).
Aleph provides a fully featured client for `mcnode` named `mcclient` and we use it in
the examples below.

Once your node is up and running you can check its status:

```
$ mcclient id
Peer ID: QmeBkfxcaBfA9pvzivRwhF2PM7sXpp4HHQbp7jfTkRCWEa
Publisher ID: 4XTTMADSKQUN3jkeZngbtuE35w9y5YnDTicVTeeji7N2Npkey
Info:
$ mcclient status
offline
```

Since our status is `offline`, we can't reach other peers, and attempting to do so will
result in an error:

```
$ mcclient listPeers
Internal Server Error
Error: Node is offline
```

Let's fix that by setting a directory server and taking our node online.

### Configuring the directory
At this point you should designate a directory for looking up peers.  We'll use the
[multiaddress][multiaddr] of the Mediachain Labs public directory,
`/ip4/52.7.126.237/tcp/9000/p2p/QmSdJVceFki4rDbcSrW7JTJZgU9so25Ko7oKHE97mGmkU6`.

```
$ mcclient config dir /ip4/52.7.126.237/tcp/9000/p2p/QmSdJVceFki4rDbcSrW7JTJZgU9so25Ko7oKHE97mGmkU6
set directory to /ip4/52.7.126.237/tcp/9000/p2p/QmSdJVceFki4rDbcSrW7JTJZgU9so25Ko7oKHE97mGmkU6
```

The directory allows you to contact other nodes that have registered themselves by setting
their status to `public`.  When contacting a remote node, we use their Peer ID, which is an
encoded [multihash][multihash] that's derived from the node's public key.  The directory
maps Peer IDs to routable addresses that our node can use to connect to others.

Setting the directory does not make your node visible to others; for that we need to set our
status to `public`.  We'll do that in a little while; first lets take our node online so that
it can query the directory and contact other peers.

## Taking the node online
Set the node's status to `online`:

```
$ mcclient status online
status set to online
```

Now we can query the directory for the list of registered peers:

```
$ mcclient listPeers
...
QmeiY2eHMwK92Zt6X4kUUC3MsjMmVb2VnGZ17DhnhRPCEQ
...
```

And learn more about them:

```
$ mcclient id QmeiY2eHMwK92Zt6X4kUUC3MsjMmVb2VnGZ17DhnhRPCEQ
Peer ID: QmeiY2eHMwK92Zt6X4kUUC3MsjMmVb2VnGZ17DhnhRPCEQ
Publisher ID: 4XTTM4K8sqTb7xYviJJcRDJ5W6TpQxMoJ7GtBstTALgh5wzGm
Info: Metadata for CC images from DPLA, 500px, and pexels; operated by Mediachain Labs.
```

We can also get the `Info` string alongside the directory listing with `mcclient listPeers --info`:

```
$ mcclient listPeers --info
QmTVsocFCSEdPyM8dZ734GRhjvvmYyL9fShyezVkbPj17E -- Curated Museum Metadata; Operated by Mediachain Labs
QmeiY2eHMwK92Zt6X4kUUC3MsjMmVb2VnGZ17DhnhRPCEQ -- Metadata for CC images from DPLA, 500px, and pexels; operated by Mediachain Labs.
QmZ6dckUhRouVr6AsBTpK6vMLVpcz1KAeJAJVQEZQ5gCek -- Metadata for CC images from flickr; operated by Mediachain Labs.
...
```

## Next steps

See the [Basic Operations guide][basic-ops] to learn how to [query][basic-ops-query],
[publish data][basic-ops-publish], [merge remote data][basic-ops-merge] and make your node
[publicly accessible][basic-ops-public].


[multiaddr]: https://github.com/multiformats/multiaddr
[multihash]: https://github.com/multiformats/multihash
[basic-ops]: {{site.baseurl}}{% link _docs/basic-operations.md %}
[basic-ops-query]: {{site.baseurl}}{% link _docs/basic-operations.md %}#query
[basic-ops-publish]: {{site.baseurl}}{% link _docs/basic-operations.md %}#publishing-statements
[basic-ops-merge]: {{site.baseurl}}{% link _docs/basic-operations.md %}#merge
[basic-ops-public]: {{site.baseurl}}{% link _docs/basic-operations.md %}#going-public
