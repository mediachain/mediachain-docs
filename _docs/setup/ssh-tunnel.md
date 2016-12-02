---
title: Controlling a remote node with SSH Tunneling
layout: page
category: setup
order: 5
---

* TOC
{:toc #table_of_contents}

To control a [concat][concat] `mcnode` running on a remote machine, you can configure `mcclient` to
automatically create a secure "tunnel" to the remote machine using SSH.   This document assumes that
you have `mcclient` installed.  If not, see the [install instructions]({{ site.baseurl }}{% link _docs/setup/install.md %}).

All you need is a configuration file to let `mcclient` know how to reach your node and log in. If
you used  [Mediachain Deploy]({{ site.baseurl }}{% link _docs/setup/cloud-deploy.md %}) to create your server, the credentials file that you
downloaded at the end of the Deploy process can be used directly to set up the tunnel.
Let's say you've saved the credentials file for a node with the IP address `67.205.161.186` to
a `~/mediachain/mediachain_node_67.205.161.186.json`.  Just pass in that path with the `--sshConfig`
flag when you run `mcclient`:

```
$ mcclient --sshConfig ~/mediachain/mediachain_node_67.205.161.186.json id
Peer ID: QmQD6ZkWFQWsAt3hA71V9zjCyphQ7q62sWAYwPrUMXTnaN
Publisher ID: 4XTTMBoNJsmDEhKYRypFbcUwxvny5w57syL6KDbhUFX7yCnjp
Info:
```

## Creating an SSH configuration file manually

To tunnel to a server that you've set up yourself without Mediachain Deploy, you can create
a simple JSON file containing an object with these fields:

- `host`: the IP address or hostname of the remote server
- `username`: the username of the user running the `mcnode` daemon
- either `password` or `privateKey`
    - if `privateKey` is given, it should be the full contents of your SSH private key file as a
    JSON string, not a file path.  Be careful about newlines, which need to be encoded as `\n` in
    the JSON string.  If you have the excellent [jq](https://stedolan.github.io/jq) installed,
    you can use this one-liner to get the correct formatting: `cat ~/.ssh/id_rsa | jq -Rs .`

#### Optional fields

If your remote `mcnode` has its API server bound to a non-standard port, you can use the
`dstPort` field, e.g. `dstPort: 6789`. If you do use a non-standard `dstPort`, it's a good
idea to also set `localPort: 9002`, which will bind the tunnel to the default local port of 9002,
so that `mcclient` can find the tunnel at the default location.  


## Manually creating a tunnel with SSH

It's also possible to use the `ssh` tool to create the tunnel, instead of letting `mcclient` create
it for you.  To do so, open a new terminal and enter this command, replacing `<username>` with the
remote username and `<ip-or-hostname>` with the address of your remote server:

```
ssh -nNT -L 9002:localhost:9002 <username>@<ip-or-hostname>
```

After successfully logging in, there won't be any output; that's normal! Just open a new terminal
and use `mcclient` as if `mcnode` were running on your local machine.

[concat]: https://github.com/mediachain/concat
