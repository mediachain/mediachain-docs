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
you want to publish some data and make it available to the public.  If you've [deployed your node
to the cloud]({{ site.baseurl }}{% link _docs/cloud-deploy.md%}), everything will work just the same, but you'll need to
[set up an SSH tunnel to control your node]({{ site.baseurl }}{% link _docs/ssh-tunnel.md%}).

## First steps

### Starting a new node
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

### Bringing the node online
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

### Taking the node online
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

## Basic Operations
The basic mediachain operations are queries, local or remote,  and merges of remote datasets.

### Query
Here, we query the namespaces for statements in the discovered peer's datastore and take a small
sample from the `images.dpla` namespace:

```
$ mcclient query -r QmeiY2eHMwK92Zt6X4kUUC3MsjMmVb2VnGZ17DhnhRPCEQ "SELECT namespace FROM *"
'images.500px'
'images.dpla'
'images.pexels'

$ mcclient query -r QmeiY2eHMwK92Zt6X4kUUC3MsjMmVb2VnGZ17DhnhRPCEQ "SELECT COUNT(*) FROM images.dpla"
3738109

$ mcclient query -r QmeiY2eHMwK92Zt6X4kUUC3MsjMmVb2VnGZ17DhnhRPCEQ "SELECT * FROM images.dpla LIMIT 5"
{ id: '4XTTM4K8sqTb7xYviJJcRDJ5W6TpQxMoJ7GtBstTALgh5wzGm:1476970964:0',
  publisher: '4XTTM4K8sqTb7xYviJJcRDJ5W6TpQxMoJ7GtBstTALgh5wzGm',
  namespace: 'images.dpla',
  body:
   { Body:
      { Simple:
         { object: 'Qma1LUdw5PAjfuZLXCbT5Qm5xnQFLkEejyXbLuvcKinF8K',
           refs: [ 'dpla_1349ede833fa9417b6f55be6bb402f6d' ] } } },
  timestamp: 1476970964,
  signature: 'mZMSMdMrahd40uyJChHAFLMvB5diR8qh9QI2kw0XUR7HxTo6sh2jtCzHZVBaKnOa7w9QkSrPdEU8qqfAsBEiDA==' }
...  
```

### Merge
We can merge remote datasets using a query; the node will merge in
the statements selected by the query and associated metadata:

```
$ mcclient merge QmeiY2eHMwK92Zt6X4kUUC3MsjMmVb2VnGZ17DhnhRPCEQ "SELECT * FROM images.dpla LIMIT 5"
merged 5 statements and 5 objects
```

The statements and the associated metadata our now stored in the local node:

```
$ mcclient query "SELECT * FROM images.dpla"
{ id: '4XTTM4K8sqTb7xYviJJcRDJ5W6TpQxMoJ7GtBstTALgh5wzGm:1476970964:0',
  publisher: '4XTTM4K8sqTb7xYviJJcRDJ5W6TpQxMoJ7GtBstTALgh5wzGm',
  namespace: 'images.dpla',
  body:
   { Body:
      { Simple:
         { object: 'Qma1LUdw5PAjfuZLXCbT5Qm5xnQFLkEejyXbLuvcKinF8K',
           refs: [ 'dpla_1349ede833fa9417b6f55be6bb402f6d' ] } } },
  timestamp: 1476970964,
  signature: 'mZMSMdMrahd40uyJChHAFLMvB5diR8qh9QI2kw0XUR7HxTo6sh2jtCzHZVBaKnOa7w9QkSrPdEU8qqfAsBEiDA==' }
...

The following `getData` command is targeting our local node, not the remote peer.  Since the
`object` (referenced in the statement above) was successfully merged in, we get the data we want:

$ mcclient getData Qma1LUdw5PAjfuZLXCbT5Qm5xnQFLkEejyXbLuvcKinF8K
{ orientation: null,
  dedupe_hsh: '4e0783f1e8f41bac',
  licenses: [ { details: null } ],
  native_id: 'dpla_http://dp.la/api/items/1349ede833fa9417b6f55be6bb402f6d',
  keywords: [],
  date_created_original: null,
  title: [ 'Page 91' ],
  camera_exif: {},
  source: { url: 'https://dp.la/', name: 'dpla' },
  transient_info: { score_hotnessviews: null, likes: null },
  date_captured: null,
  location: { lat_lon: null, place_name: [] },
  attribution: null,
  description: 'kada',
  source_tags: [ 'dp.la' ],
  date_created_at_source: null,
  providers_list:
   [ { name: 'dpla' },
     { name: 'University of Southern California. Libraries' } ],
  date_source_version: null,
  sizes:
   [ { bytes: null,
       height: 120,
       uri_external: 'http://digitallibrary.usc.edu/utils/getthumbnail/collection/p15799coll126/id/7763',
       width: 66,
       content_type: 'image/jpeg',
       dpi: null } ],
  source_dataset: 'dpla',
  artist_names: [],
  url_direct: { url: 'http://digitallibrary.usc.edu/utils/getthumbnail/collection/p15799coll126/id/7763' },
  derived_qualities:
   { medium: null,
     predicted_tags: null,
     has_people: null,
     colors: null,
     general_type: null,
     time_period: null },
  url_shown_at: { url: 'http://digitallibrary.usc.edu/cdm/ref/collection/p15799coll126/id/7763' },
  aspect_ratio: 0.55 }

```

If you decide you no longer want to keep the statements you merged in your local store,
you can delete them:

```
$ mcclient delete "DELETE FROM images.dpla"
Deleted 5 statements
```

### Publishing Statements

You can publish statements to your local node by creating json objects with the metadata
you want to publish.

For example, let's publish hello world in a couple different variations:

```
$ cat /tmp/hello.json
{"id": "hello_1", "hello": "world"}
{"id": "hello_2", "hola": "mundo"}
$ mcclient publish --idFilter '.id' scratch.hello /tmp/hello.json
statement id: 4XTTMADSKQUN3jkeZngbtuE35w9y5YnDTicVTeeji7N2Npkey:1477063161:0 -- body: QmZDxgNgUT1J3rgjvnGjoxoA5efGNSN9Qvhq4FpvefmwnA -- refs: ["hello_1"]
statement id: 4XTTMADSKQUN3jkeZngbtuE35w9y5YnDTicVTeeji7N2Npkey:1477063161:1 -- body: QmcdACgobENfs7vuD2CpQGi8RdEkNyAmbmyqVvrA7Z57xu -- refs: ["hello_2"]
All statements published successfully
```

The statements and the metadata are now stored by the local node:

```
$ mcclient query 'SELECT * FROM scratch.hello'
{ id: '4XTTMADSKQUN3jkeZngbtuE35w9y5YnDTicVTeeji7N2Npkey:1477063161:0',
  publisher: '4XTTMADSKQUN3jkeZngbtuE35w9y5YnDTicVTeeji7N2Npkey',
  namespace: 'scratch.hello',
  body:
   { Body:
      { Simple:
         { object: 'QmZDxgNgUT1J3rgjvnGjoxoA5efGNSN9Qvhq4FpvefmwnA',
           refs: [ 'hello_1' ] } } },
  timestamp: 1477063161,
  signature: 'Q79Vgp7bl4J7rJo3DOmtueLGqHQpG3lpSyiXbTqX60oOdbPdju06m4epiQNrGLarfCsj2Opa0psX6EWm6BJkDw==' }
{ id: '4XTTMADSKQUN3jkeZngbtuE35w9y5YnDTicVTeeji7N2Npkey:1477063161:1',
  publisher: '4XTTMADSKQUN3jkeZngbtuE35w9y5YnDTicVTeeji7N2Npkey',
  namespace: 'scratch.hello',
  body:
   { Body:
      { Simple:
         { object: 'QmcdACgobENfs7vuD2CpQGi8RdEkNyAmbmyqVvrA7Z57xu',
           refs: [ 'hello_2' ] } } },
  timestamp: 1477063161,
  signature: 'bN7mAW3JKJAHtInSGM08IXoUx//qFJpySaCWPEe8P6Pm46XxoOrepG2Q2KrIQ5bs5oLKJkU7qHqFI2dQaHETCQ==' }

$ mcclient getData QmZDxgNgUT1J3rgjvnGjoxoA5efGNSN9Qvhq4FpvefmwnA
{ id: 'hello_1', hello: 'world' }
```

## Going Public

In order to distribute statements stored in your node, it needs to become
discoverable and accessible to other nodes. This happens by taking the node
`public`, which registers with the directory.

Before you can take your node public however, you need to ensure that it is
reachable in the network. By default, `mcnode` binds its p2p interface in port 9001
for all interfaces, so if your node is directly connected to the Internet (eg in a vps
host), you don't have to do anything.

However, chances are your node is behind a NAT and you will need to
[configure it for NAT traversal]({{ site.baseurl }}{% link _docs/nat.md %}).

Once your node's p2p port is reachable from the internet, you're almost ready to take it
public.  As a final preparation for going public, you should add a short description
about your node and its contents:

```
$ mcclient config info "Metadata from ...; Operated by ..."
```  

Then your node should be ready to take it public:

```
$ mcclient status public
```

This will register the node with the configured directory, which will make it visible
to other nodes through `mcclient listPeers`.

If you want to take your node back offline, you can simply do so with `status offline`:

```
$ mcclient status offline
```

[multiaddr]: https://github.com/multiformats/multiaddr
[multihash]: https://github.com/multiformats/multihash
