---
title: Using Mediachain Deploy
layout: page
category: setup
order: 2
---

* TOC
{:toc #table_of_contents}

[Mediachain Deploy][mc-deploy] is a one-click tool for creating and configuring a mediachain
node in the cloud.  Deploy uses DigitalOcean's API to automatically create a low-cost ($5/month)
private server on your behalf.  All you need is a DigitalOcean account and an API key, and
Deploy will install `mcnode`, the main mediachain node software from the [concat project][concat].

For more information, see the [blog post][blog-post] introducing Deploy, or just
[go to the Deploy site][mc-deploy] and give it a try.  If you sign up for your DigitalOcean
account with [our referral code][do-referral], you'll get $10 of credit, which is enough to
run your node 24/7 for the first two months.

## Controlling your node

Once the Deploy process is complete, make sure to click the button to "Save Your Credentials!"

With [mcclient installed on your local computer]({{ site.baseurl }}{% link _docs/setup/install.md %}#installing-aleph), you can use the
credentials file to [set up a secure tunnel]({{ site.baseurl }}{% link _docs/setup/ssh-tunnel.md %}) to the remote server and control it as
if it were running on your local machine:

```
$ mcclient --sshConfig ~/mediachain/mediachain_node_67.205.161.186.json id
Peer ID: QmQD6ZkWFQWsAt3hA71V9zjCyphQ7q62sWAYwPrUMXTnaN
Publisher ID: 4XTTMBoNJsmDEhKYRypFbcUwxvny5w57syL6KDbhUFX7yCnjp
Info:
```

Be sure to replace `~/mediachain/mediachain_node_67.205.161.186.json` with the location of
your saved credentials file!

[concat]: https://github.com/mediachain/concat
[mc-deploy]: http://deploy.mediachain.io
[blog-post]: https://blog.mediachain.io/mediachain-deploy-1c625e69d124#.5wjucqj83
[do-referral]: https://m.do.co/c/a688ee6191f9
