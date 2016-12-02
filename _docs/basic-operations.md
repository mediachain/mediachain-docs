---
title: Basic Operations
layout: page
category: usage
order: 1
---

* TOC
{:toc #table_of_contents}

The basic mediachain operations are queries, local or remote, and merges of remote datasets.
Let's see some examples. This guide assumes you've followed the
[installation][install] and [getting started][getting-started] guides, and have
[configured your node to use the Mediachain Labs directory server][gs-config-dir].

## Discovery

Remote peers are discovered by asking the directory server for a list of all registered peers:

```
$ mcclient listPeers
QmeiY2eHMwK92Zt6X4kUUC3MsjMmVb2VnGZ17DhnhRPCEQ
# ...

```

It's helpful to provide the `--info` flag to get some more information about each peer:

```
$ mcclient listPeers --info
QmTVsocFCSEdPyM8dZ734GRhjvvmYyL9fShyezVkbPj17E -- Curated Museum Metadata; Operated by Mediachain Labs
QmeiY2eHMwK92Zt6X4kUUC3MsjMmVb2VnGZ17DhnhRPCEQ -- Metadata for CC images from DPLA, 500px, and pexels; operated by Mediachain Labs.
QmZ6dckUhRouVr6AsBTpK6vMLVpcz1KAeJAJVQEZQ5gCek -- Metadata for CC images from flickr; operated by Mediachain Labs.
# ...
```

To make your own node discoverable, see the [Going Public](#going-public) section below.

## Query
You can query a mediachain node using [MCQL]({{site.baseurl}}{% link _docs/mcql.md %}), the
Mediachain Query Language.  MCQL is very similar to SQL, with mediachain namespaces taking
place of SQL tables.

Use the `mcclient query` command to send a query to your own node (either on your local
machine or a remote server via an [ssh tunnel]({{site.baseurl}}{% link _docs/ssh-tunnel.md %})), or
any remote peer that your node has discovered through the directory.

Let's start with a remote query to a node run by Mediachain Labs, discovered in the `listPeers`
invocation above.

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

### Query by Well-Known Identifier

Since many metadata objects already have external identifiers, mediachain uses the concept of a
[Well-Known Identifier][arch-wki] or WKI to index statements within a node.

In the query example above, the WKI is stored in the `refs` field of the statement body.  We can
ask for just statements for that one identifier with a `WHERE` clause:

```
$ mcclient query -r QmeiY2eHMwK92Zt6X4kUUC3MsjMmVb2VnGZ17DhnhRPCEQ "SELECT * FROM images.dpla WHERE wki = dpla_1349ede833fa9417b6f55be6bb402f6d"
{ id: '4XTTM4K8sqTb7xYviJJcRDJ5W6TpQxMoJ7GtBstTALgh5wzGm:1476970964:0',
  ... }
```

## Merge

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

The following `data get` command is targeting our local node, not the remote peer.  Since the
`object` (referenced in the statement above) was successfully merged in, we get the data we want:

$ mcclient data get Qma1LUdw5PAjfuZLXCbT5Qm5xnQFLkEejyXbLuvcKinF8K
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

## Delete

If you decide you no longer want to keep the statements you merged in your local store,
you can delete them:

```
$ mcclient delete "DELETE FROM images.dpla"
Deleted 5 statements
```

Note that `DELETE` queries must use the `mcclient delete` command; if you try to issue one with
`mcclient query` you'll get an error.  

Like queries, you can use a `WHERE` clause to constrain the deletion:

```
$ mcclient delete "DELETE FROM images.dpla WHERE wki = dpla_1349ede833fa9417b6f55be6bb402f6d"
Deleted 1 statement
```

If you delete a large number of statements, it may be beneficial to run a
[garbage collection][advanced-gc] pass on your node's datastore to reclaim space.

## Publishing Statements

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

The `--idFilter` flag above is used to extract a

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
[configure it for NAT traversal][nat].

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

[install]: {{site.baseurl}}{% link _docs/install.md %}
[nat]: {{ site.baseurl }}{% link _docs/nat.md %}
[getting-started]: {{site.baseurl}}{% link _docs/getting-started.md %}
[gs-config-dir]: {{site.baseurl}}{% link _docs/getting-started.md %}#configuring-the-directory
[advanced-gc]: {{site.baseurl}}{% link _docs/advanced-operations.md %}#garbage-collection
[arch-wki]: {{site.baseurl}}{% link _docs/statements-and-objects.md %}#well-known-identifiers
