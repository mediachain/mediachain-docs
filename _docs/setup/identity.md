---
title: Linking Your Identity
layout: page
category: setup
order: 2
---

* TOC
{:toc #table_of_contents}

## Blockstack
First, follow the [documentation](https://blockstack.org/docs) to set up a Blockstack identity.
```
# generate a new mediachain identity
% mcid id
Enter passphrase: **************
{"keyId":"QmckUW1W9nhmdidAEmMA1R8AdfGEc8uM2jmPwpZ5ktp44Z","key":"CAESIJXCFQpzGJayioeElDxGHRRM++9UQ/h/X6a8DTtD1PEE"}

# write the identity to your blockstack profile
TODO

# generate a node manifest describing your peer
% mcclient manifest self > /tmp/mf.json

# sign the manifest attesting that you’re “arkadiy” on blockstack
% mcid sign blockstack:denisnazarov /tmp/mf.json > /tmp/smf.json
Enter passphrase: **************

# verify that the blockstack user “denisnazarov” presents a valid mediachain identity
% mcid verify /tmp/smf.json
OK

# add the manifest to our node
% mcclient manifest add /tmp/smf.json
Manifest added successfully
```

## Keybase
First, follow the [documentation](https://keybase.io/download) to set up a Keybase identity. 
Join our [Slack](http://slack.mediachain.io) if you need an invite!
```
# generate a new mediachain identity
% mcid id
Enter passphrase: **************
{"keyId":"QmckUW1W9nhmdidAEmMA1R8AdfGEc8uM2jmPwpZ5ktp44Z","key":"CAESIJXCFQpzGJayioeElDxGHRRM++9UQ/h/X6a8DTtD1PEE"}

# write the identity to your public keybase directory
% mcid id > /keybase/public/arkadiy/mediachain.json

# generate a node manifest describing your peer
% mcclient manifest self > /tmp/mf.json

# sign the manifest attesting that you’re “arkadiy” on keybase
% mcid sign keybase:arkadiy /tmp/mf.json > /tmp/smf.json
Enter passphrase: **************

# verify that the keybase user “arkadiy” presents a valid mediachain identity
% mcid verify /tmp/smf.json
OK

# add the manifest to our node
% mcclient manifest add /tmp/smf.json
Manifest added successfully
```
