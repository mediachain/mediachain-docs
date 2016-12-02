---
title: Installing Mediachain
layout: page
category: setup
order: 1
---

* TOC
{:toc #table_of_contents}

Mediachain is comprised of two main software projects, [concat][concat] and [aleph][aleph].
If we were to split them along the popular "porcelain" / "plumbing" axis, aleph would be
the porcelain to concat's plumbing.  The storage, indexing and query of metadata is all
handled by concat's `mcnode` program, and the concat project also includes the reference
directory server implementation `mcdir`.

Controlling a running `mcnode` is done by issuing HTTP requests to an embedded server that
listens on the `localhost` interface.  While it's possible to control `mcnode` directly with `curl`
or a similar tool, aleph's `mcclient` tool is much friendlier and better suited to the job.

The aleph project also includes an interactive REPL for querying a local or remote `mcnode`,
which provides a flexible javascript environment for exploring the mediachain network.

Which you should install depends on your goals.  If you want to control a node that you've
[deployed to the cloud][deploy], all you need to do is [install aleph](#installing-aleph).

If you want to run a node on your local machine, you should [install concat](#installing-concat), then
[install aleph](#installing-aleph).

Be sure to check out the [Getting Started Guide]({{ site.baseurl }}{% link _docs/getting-started.md %}) after installation to learn
how to use mediachain to query, publish and share data with the network!

## Installing concat

Concat's `mcnode` program is the main mediachain node application, responsible for data storage,
transmission, and query.  It can be easily [installed to the cloud][deploy] with one click, but
if you prefer to run it on your own hardware, the installation is straightforward.

In most cases, a binary release will be the simplest way to get up and running. Find the `mcnode`
package for your platform from [the latest GitHub release page](https://github.com/mediachain/concat/releases/latest).
Linux packages will end with `-linux-amd64.tgz`, and macOS packages will end with `-darwin-amd64.tgz`.

On Windows, we currently recommend installing the linux binaries under the Windows Subsystem for Linux.
The concat README has [setup instructions](https://github.com/mediachain/concat#running-on-windows).

To get the latest pre-release code, or if you want to run `mcnode` on a platform without a binary
release (FreeBSD, Linux on ARM, etc), follow the instructions from concat's README for
[installing from source](https://github.com/mediachain/concat#installing-from-source).

## Installing aleph

### Dependencies
The `aleph` project is written in javascript on the Node.js platform, and it requires node version
6 or greater.  

#### Linux

Linux users will need to have a modern c++ compiler installed to build the native dependencies.
You'll also need the development headers for openssl for the crypto libraries.
On Ubuntu and other debian-based distros, you should be able to do:

```
$ sudo apt-get install build-essential g++ libssl-dev
```

#### macOS

Users on macOS will need the Xcode command line tools installed.  This can be done from within
Xcode (under Preferences -> Downloads), or
[as a standalone package](http://railsapps.github.io/xcode-command-line-tools.html).

#### Windows

We're [working on](https://github.com/mediachain/aleph/issues/105) "real" Windows support, but for
now we recommend using the
[Windows Subsystem for Linux](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide),
which creates a minimal Ubuntu system that runs under Windows 10 without virtualization. This supports
native linux binaries, and will allow you to install the Linux versions of node.js, git, and other
aleph dependencies.

To install aleph's dependencies on the Windows Subsystem for Linux:

- [Enable the WSL](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide)
- Open a command prompt or PowerShell, then run `bash`.  This may take a few minutes to prepare the
linux root filesystem.  

The following steps should be done from within bash:

- Prepare the system to install node.js: `curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -`
- Install dependencies: `sudo apt-get install build-essential g++ libssl-dev git nodejs`

Now you should be able to install aleph within bash with the instructions below.

### Installation
The simplest way to get started is to globally install the latest release from npm:

```
$ npm install -g aleph
```

Depending on how node is installed on your system, you may need to run the above command with `sudo`
or as a user with administrator privileges.  For other install methods, see
[the aleph README](https://github.com/mediachain/aleph/blob/master/README.md#installation).

Aleph provides an `mcclient` program for controlling a concat node, and an `aleph` command for
running a lightweight javascript node.  In the instructions below, we'll discuss using `mcclient` to
control a concat node.  More information about the `aleph` command is available in
[the aleph README](https://github.com/mediachain/aleph/blob/master/README.md#aleph).

### Configure `mcclient`

By default `mcclient` is configured to talk to `mcnode` at `http://localhost:9002`, which is the
default address for the `mcnode` control API server.  If `mcnode` is running on your local machine
(see [above](#installing-mcnode) for instructions), you shouldn't need to configure anything.

If you're controlling a node you've [deployed to the cloud][deploy] or installed yourself on a
remote machine, you need to first set up a secure "tunnel", that will forward traffic from your
local machine to the remote server.  See the [SSH tunnel instructions]({{ site.baseurl }}{% link _docs/ssh-tunnel.md %}) for more
information on setting up the tunnel.  Once it's running, you'll be able to control your remote
node as if it were running on your local machine.

### Usage

See the [Getting Started Guide]({{ site.baseurl }}{% link _docs/getting-started.md %}) to learn how to take your node online,
run queries, publish data, and more.

[concat]: https://github.com/mediachain/concat
[aleph]: https://github.com/mediachain/aleph
[deploy]: cloud-deploy.md
