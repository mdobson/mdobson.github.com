---
layout: post
title: Creating a Custom CoreOS box with Vagrant and AWS 
---

Creating a custom CoreOS box with vagrant can be troublesome. After creating a build with packer, and trying to use vagrant to start the virtualbox odds are you will come upon the following error.

```bash
The following SSH command responded with a non-zero exit status.
Vagrant assumes that this means the command failed!

mv /tmp/etcd-cluster.service /media/state/units/

Stdout from the command:



Stderr from the command:

mv: cannot move '/tmp/etcd-cluster.service' to '/media/state/units/': No such file or directory
``` 

What is happening is vagrant is trying to patch an old version of CoreOS to enable network configurations to take place. The team at CoreOS has monkey patched these configurations using custom plugins on their box builds, but the standard plugins for CoreOS in vagrant still try and conform to the old media units.

Here is the fix. 

For a custom build of CoreOS go to their repo and download the following plugins.

- [`change_host_name.rb`](https://github.com/coreos/coreos-overlay/raw/master/coreos-base/oem-vagrant/files/box/change_host_name.rb)
- [`configure_networks.rb`](https://github.com/coreos/coreos-overlay/raw/master/coreos-base/oem-vagrant/files/box/configure_networks.rb)
- [`base_mac.rb`](https://github.com/coreos/coreos-overlay/raw/master/coreos-base/oem-vagrant/files/box/base_mac.rb)

Then at the top of your vagrant file place these require statements.

```ruby
require_relative 'change_host_name.rb'
require_relative 'configure_networks.rb'
require_relative 'base_mac.rb'
```

This will monkey patch the network configuration to function like the stock vagrant box available off the CoreOS website.
