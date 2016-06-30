---
layout: post
title:  "AutoYaST + Salt et al."
date:   2016-06-30 09:30:00 +0100
categories: YaST
tags: YaST AutoYaST HackWeek
---
Recently, during an internal workshop at SUSE, I decided to write a
proof of concept to combine AutoYaST and tools like
[Salt](https://saltstack.com/) or [Puppet](https://puppet.com). As it
was fun for me, I decided to work a little bit more on it during
[this Hack Week](https://hackweek.suse.com/14/projects/1372).

The idea is pretty simple but, before digging deeper, I would like to
start describing (briefly) how AutoYaST works.

## How AutoYaST works

AutoYaST installs and configures a SUSE/openSUSE Linux system in an
unattended way according to a _profile_ (a XML-based
specification). You can check the
[AutoYaST handbook](https://www.suse.com/documentation/sles-12/singlehtml/book_autoyast/book_autoyast.html)
for further details.

The installation/configuration is splitted in two different phases (known
as _stages_):

* During the first one, it will take care of partitioning, software
  installation, bootloader configuration, etc. The _harder_ stuff
  IMHO.
* After rebooting, AutoYaST will enter in the second stage, configuring
  several things like network, NTP, Samba, etc.

This tool is **pretty flexible** and have a ton of options. Additionally,
AutoYaST is also capable to configure already installed systems,
although that's not the main use case.

## Software configuration management

Software configuration management tools, like Salt, Puppet or
[Chef](https://www.chef.io/), to name a few, have gained a lot of
traction. So, from my POV, it makes sense to enable AutoYaST to play
nicely with them.

Fortunately, AutoYaST has support for running scripts in several
points of the installation, so it's not that hard to write profiles to
use those tools.

However, would not be nice to have 1st class citizen support? :)

## YaST CM

[Yast CM](https://github.com/imobachgs/yast-cm) is a module that
(somehow) integrates AutoYaST and Software Configuration Managment
tools. Currently, Puppet and Salt are supported.

During AutoYaST 2nd stage, this module will take care of:

* Install needed packages (if you're using Salt, you should add the
  [SUSE/openSUSE repository](https://docs.saltstack.com/en/latest/topics/installation/suse.html)).
* Retrieve authentication keys.
* Update Salt/Puppet configuration if needed.
* Apply states/recipes.

![Running provisioner](/images/yast-cm-20160630.png)

Consider the following AutoYaST profile snippet:

```xml
<scm>
  <type>salt</type> <!-- you can use "puppet" -->
  <master>my-salt-server.example.net</master>
  <attempts config:type="integer">5</attempts>
  <timeout config:type="integer">10</timeout>
  <keys_url>usb:/</keys_url> <!-- you can use HTTP, FTP... -->
</scm>
```

During the 2nd stage:

* Authentication keys will be retrieved from an USB stick.
* Salt will be configured to connect to `my-salt-server.example.net`.
* The system will be configured using Salt.

If you prefer not to use a Salt/Puppet server, the module can also operate
in _masterless_ mode:

```xml
<scm>
  <type>salt</type> <!-- you can use "puppet" -->
  <config_url>http://myserver.example.net/states.tgz</config_url>
  <attempts config:type="integer">3</attempts>
</scm>
```

In this second case, the configuration will be fetched from a
HTTP server and applied locally.

## Current state

This module is just an experiment and a lot of work remains to be
done, like for example:

* Add feedback (currently you should check logs to know what's happening).
* Improve (a lot) error handling.
* Find a good name.
* A lot of testing.
* Refactor, refactor, refactor!

## About Hack Week

Hack Week is a really nice SUSE tradition since 2007. During a week,
we can experiment, learn and work in any project we want. A
developer's dream!

But let me make this clear: Hack Week is open for cooperation and
the community is invited to join us! So, what are you waiting for?
