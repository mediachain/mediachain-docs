---
title: Configuring NAT Traversal
layout: page
category: setup
order: 4
---

To take your node public, the peer-to-peer interface must be reachable from
the public internet.  If you're behind a home router or other firewall, you'll
likely need to configure NAT traversal to make the p2p port accessible.

The default `mcnode` p2p port is 9001, which we want to make available to the
outside world.

First, take the node offline because NAT config changes only take effect when the
node goes online:
```
$ mcclient status offline
```

If you are behind a home router which supports upnp port mapping, you can
set your NAT configuration to "auto":
```
$ mcclient config nat auto
```

If you are behind multiple NATs or behind a firewall that doesn't support port mapping
(eg an AWS instance), then you need to manually configure your network to route traffic
to the p2p port in your host.
If you mapped the port transparently (so that the port numbers are the same inside and
outside your local network), you can set your NAT configuration to "*", which
will automatically detect the IP address and advertise the locally bound port:
```
$ mcclient config nat "*"
```

If you have mapped a different port to your host interface, then you need to specify
it explicitly:
```
$ mcclient config nat "*:port"
```
