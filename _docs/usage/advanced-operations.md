---
title: Advanced Operations
layout: page
category: usage
order: 1000
---

* TOC
{:toc #table_of_contents}

## Datastore management

### Garbage Collection

After deleting statements from your node, you can reclaim space by running a garbage collection
pass on your node's datastore.  Mediachain separates storage into a statement database (which
powers the queries), and an object datastore, which contains the metadata objects.  Since an
object may be referenced by more than one statement, it isn't safe to automatically delete objects
after a `DELETE` query.

Garbage collection removes any data objects which are no longer referenced by any statements.
Note that the node must be offline during the collection process.

```
# show the current status of the node
$ mcclient status
online

# go offline
$ mcclient status offline
status set to offline

# trigger GC
$ mcclient data gc
Garbage collected 5 objects

# bring the node back online
$ mcclient status online
```
