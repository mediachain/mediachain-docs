---
title: Pushing Data to Remote Peers
layout: page
order: 10
category: usage
---

Similar to the [merge operation][basic-ops-merge], which pulls data from a remote peer into your own
node, the push operation sends data from your node to theirs.

While merges can be done from any publicly accessible peer, pushing data to a remote peer requires
prior authorization.  This guide will walk through the authorization and push commands so you can
send your data to a peer directly, without requiring them to initiate a merge.

## Authorization

Before a node can push data to a remote peer, that peer needs to authorize the node to write to
the namespaces for the statements it wants to push.

The `mcclient` tool has commands for showing the current set of authorizations, granting new
authorizations, and revoking existing authorizations.

A node's authorization table starts out empty:

```
$ mcclient auth show
{}
```

We can grant a peer access to a set of namespaces by using its peer id:

```
$ mcclient auth grant QmXY1nyb6sbXgdAjEjDHcqXfpmmwjhj3S1KPSu94grp1xo 'museums.*'
Granted authorizations for peer QmXY1nyb6sbXgdAjEjDHcqXfpmmwjhj3S1KPSu94grp1xo:
[
  "museums.*"
]
```

The `*` wildcard allows the peer with id `QmXY1nyb6sbXgdAjEjDHcqXfpmmwjhj3S1KPSu94grp1xo` to push
data to any namespace nested within the `museums` root namespace.  Note the single quotes - this is
to prevent the unix shell from interpreting the `*` as a file glob.  

Now we can see that peer in the authoriazation list:

```
$ mcclient auth show
{
  "QmXY1nyb6sbXgdAjEjDHcqXfpmmwjhj3S1KPSu94grp1xo": [
    "*",
  ]
}
```

If we need to be more granular, we can list namespaces individually:

```
mcclient auth grant QmXY1nyb6sbXgdAjEjDHcqXfpmmwjhj3S1KPSu94grp1xo museums.moma.artists museums.moma.artworks
Granted authorizations for peer QmXY1nyb6sbXgdAjEjDHcqXfpmmwjhj3S1KPSu94grp1xo:
Granted authorizations for peer QmXY1nyb6sbXgdAjEjDHcqXfpmmwjhj3S1KPSu94grp1xo:
[
  "museums.moma.artists",
  "museums.moma.artworks"
]
```

**Important:** each invocation of `mcclient auth grant` *replaces* any existing authorizations for
that peer.

We can revoke the peer's authorizations if we no longer want them to have any access:

```
$ mcclient auth revoke QmXY1nyb6sbXgdAjEjDHcqXfpmmwjhj3S1KPSu94grp1xo
Revoked authorization for QmXY1nyb6sbXgdAjEjDHcqXfpmmwjhj3S1KPSu94grp1xo.

$ mcclient auth show
{}
```

## Pushing Data

We can push data with the `mcclient push` command, which accepts the id of the peer we want to
push to and an [MCQL][mcql] query.  The results of the query are sent to the remote peer, which
will accept them if we're authorized.

Let's assume that our local node is authorized to push to the "museum node" hosted by Mediachain
Labs, whose peer id is `QmTVsocFCSEdPyM8dZ734GRhjvvmYyL9fShyezVkbPj17E`.  The authorization is
set to `museums.tate.*`, so we're authorized for any sub-namespace within the Tate hierarchy.


We have some data locally in the `museums.tate.artists` and `museums.tate.artworks` namespaces:

```
$ mcclient query 'SELECT COUNT(*) FROM museums.tate.*'
70740
```

We can push it to `QmTVsocFCSEdPyM8dZ734GRhjvvmYyL9fShyezVkbPj17E` like this:

```
$ mcclient push QmTVsocFCSEdPyM8dZ734GRhjvvmYyL9fShyezVkbPj17E 'SELECT * FROM museums.tate.*'
Pushed 70740 statements and 70742 objects
```

If we try it again, we see that nothing was sent, since the remote peer already has everything:

```
$ mcclient push QmTVsocFCSEdPyM8dZ734GRhjvvmYyL9fShyezVkbPj17E 'SELECT * FROM museums.tate.*'
Pushed 0 statements and 0 objects
```

[basic-ops-merge]: {{site.baseurl}}{% link _docs/usage/basic-operations.md %}#merge
[mcql]: {{site.baseurl}}{% link _docs/arch/mcql.md %}
