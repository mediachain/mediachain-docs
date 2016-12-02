---
title: Statements and Objects
layout: page
category: arch
---

The `mcnode` daemon stores data in two places, statement db and the datastore.

The datastore contains the metadata per se, as CBOR objects (IPLD compatible to the best of our
ability) of unspecified schema, stored in RocksDB in point lookup mode.

The statement db contains statements about one (currently) or more metadata objects: their
publisher, namespace, timestamp and signature. Statements are protobuf objects sent over the wire
between peers to signal publication or sharing of metadata; when stored, they act as an index to the
datastore. This db is currently stored in SQLite.


### Well-Known Identifiers

TK: explain WKIs and their usage
