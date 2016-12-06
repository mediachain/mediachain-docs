---
title: Glossary
layout: page
category: arch
order: 5
---

* TOC
{:toc #table_of_contents}

Here you'll find definitions for terms used throughout the mediachain project, including this
documentation.  Filling in the glossary is an ongoing effort; please
[file an issue](https://github.com/mediachain/mediachain-docs/issues) if you come across something
that's missing or seems incomplete.

#### Data Object
A "blob" of data stored in a mediachain node's [datastore](#datastore).  This is typically a
CBOR-encoded map of metadata, similar to a JSON object.  Data objects are referred to by their
multihash in [statements](#statement), which contain the signature, namespace, and other
"meta-metadata" that contextualizes the object within the mediachain network.

#### Datastore
A content addressed on-disk repository of [data objects](#data-object) managed by a mediachain
node.  The datastore is currently implemented by [mcnode][mcnode] as a [RocksDB][rocksdb] database
whose keys are the multihash of the object's contents.

#### Multihash
An encoding for cryptographic hash digests that allows a program to identify the
algorithm used to create the hash.  Multihashes are typically represented as [base 58][base58]
strings, although they use a compact binary representation internally.  For more information,
see [https://github.com/multiformats/multihash](https://github.com/multiformats/multihash).
Multihashes used by mediachain use the SHA2-256 algorithm, which results in base 58 strings
prefixed with `Qm`, e.g. `QmTVsocFCSEdPyM8dZ734GRhjvvmYyL9fShyezVkbPj17E`.

Multihashes are used in mediachain for several purposes, most importantly as [peer ids](#peer-id)
and to identify [data objects](#data-object).

#### Namespace
A "topic" that serves to partition the statements published within the mediachain network into
related logical groupings.

Namespaces are organized in a dot-separated hierarchy that gets more specific as you descend, e.g.: `museums.moma.artists`.

#### Node
See [peer](#peer)

#### Peer
A member of the mediachain peer-to-peer network.  Technically synonymous with "node", although
the term "node" is typically used to refer to a user's local node, as opposed to a remote "peer"
in the network.

#### Peer Id
A unique, cryptographically-verifiable identifier for a mediachain peer.  The peer id is the
[multihash][multihash] of the peer's public key, and is most often used as a
[base 58][base58]-encoded string, e.g. `QmTVsocFCSEdPyM8dZ734GRhjvvmYyL9fShyezVkbPj17E`.

#### Publisher Id
The [Ed25519][ed25519] public key for a peer's signing key.  This is the keypair used to sign
statements during publication, and is distinct from the key used to derive the [peer id](#peer-id).
Typically represented as a [base 58][base58] string.

#### Statement
The main "unit of publication" within a mediachain node.  A statement contains a signed "body",
which is most often a multihash reference to a [data object](#data-object). In addition to the
body, the statement "envelope" contains  the namespace  the statement belongs to, the identity of
the publisher, links to related statements, the publication timestamp, and other contextual
information.


[rocksdb]: https://github.com/facebook/rocksdb
[mcnode]: https://github.com/mediachain/concat
[multihash]: https://github.com/multiformats/multihash
[base58]: https://en.wikipedia.org/wiki/Base58
[ed25519]: https://ed25519.cr.yp.to/
