---
layout: post
title:  "YaST2 CM gets support for Salt parametrizable formulas"
date:   2017-03-01 15:30:00 +0000
categories: YaST
tags: YaST AutoYaST HackWeek
---
Hack Week is over and it's time to share what we've built during these days.
In my case, I invested most of my time working with Duncan Mac Vicar in his
project to add support for
[Salt parametrizable formulas to YaST2](https://hackweek.suse.com/15/projects/yast-module-for-suse-manager-salt-parametrizable-formulas).

## YaST2 CM

[YaST2 CM](https://github.com/imobachgs/yast-cm) was born in 2016 as a proof of
concept to somehow integrate AutoYaST with _Software Configuration Management_
systems like [Salt](https://saltstack.com/) or [Puppet](https://puppet.com/).
The idea is that, after AutoYaST has done the main part of the installation
(partitioning, software installation, bootloader configuration, etc.), it
delegates system configuration to some other tool like Salt or Puppet.

You can read more about how it works
in the previous article I wrote about this subject:
[AutoYaST + Salt et al.](https://imobachgs.github.io/yast/2016/06/30/yast-plus-salt-et-al.html)

## SUSE Manager parametrizable formulas

To get a better insight about SUSE Manager support for parametrizable formulas,
you should
check
[Forms are the Formula for Success](https://www.suse.com/communities/blog/forms-formula-success/) at
SUSE blog. But I'll try to summarize the most important ideas.

From the
[Salt documentation](https://docs.saltstack.com/en/latest/topics/development/conventions/formulas.html):

> Formulas are pre-written Salt States. They are as open-ended as Salt States
> themselves and can be used for tasks such as installing a package, configuring,
> and starting a service, setting up users or permissions, and many other common
> tasks.

A good source of examples is the [Salt Stack repositories collection](https://github.com/saltstack-formulas)
at Github.

A key idea to formulas is that they're intended to be reused, so they are
usually configurable using the Pillar. If you look at the examples, you'll see
that most of them include a `pillar.example` with values to customize.

Parametrizable formulas improve this approach adding the specification of a form
that can be used to get the configuration values from the user. Let's see an
example taken from the aforementioned article:

```yaml
timezone:
  $type: hidden-group
  
  name:
    $type: select
    $values: ["UTC",
              "Europe/Berlin",
              "US/Mountain"]
    $default: Europe/Berlin

  utc:
    $type: boolean
    $default: True
```

Given this specification, it's pretty straighforward to build a form to get the values
from the user and turn them into data that can be included in the Pillar:

```yaml
  timezone:
    name: Europe/Berlin
    utc: True
```

## YaST2 CM forms support

Now YaST2 CM is able to use that specification to build a form and get settings from
the user. You can check [this video](https://www.youtube.com/watch?v=2em_R84XVYg)
to see it in action.

To achieve that, we improved the support for Salt _masterless_ mode. The idea is
that YaST2 will download a tarball containing all the needed files and then it
will:

* search for form specifications (`form.yml` files)
* present forms to the user in order to get the configuration parameters
* merge that information into the Pillar
* run Salt in _masterless_ mode

Here's the corresponding AutoYaST profile fragment:

```xml
<cm>
  <type>salt</type>
  <states_url>http://myserver.example.net/states.tgz</config_url>
</cm>
```

But what about the tarball layout? Perhaps it will change in the near future,
but for the time being it could contain three different directories:

* `salt/`: Salt states (including an optional `top.sls` file)
* `formulas/`: with all the formulas you want to use. YaST2 CM will search for
  `form.yml` files.
* `pillar/`: Pillar data. The formulas configuration will be written to their own
  files and merged into the (optional) `top.sls` in this directory.
  
And that's all. Pretty simple, right?

## Refactoring and clean-up

During the week, I had the chance to do an important refactor to make the module
easier to extend and support. And, by the way, I made some improvements and
fixes. I'll list the most important ones:

* The provisioner runs right after AutoYaST _autoconfiguration phase_ in order
  to avoid conflicts (for instance, while getting the libzypp lock).
* Instead of trying to unify element names in the profile for all the adapters,
  each one uses now its own names. For example, instead of using `config_url` to
  get states (Salt) or modules (Puppet) when operating in masterless mode,
  different elements will be used for each adapter: `states_url` (Salt) and
  `modules_url` (Puppet).
* Add a `pillar_url` element to Salt adapter.
* Install correct Puppet packages (using _provides_ information from libzypp).

## What's missing?

From my point of view, there are two issues we should solve in a near future:

* Although most of the code is documented, user and some high-level developer
  documentation is still missing.
* No integration testing. Almost all code is covered by unit tests but, as
  usually happens in the YaST world, there are several moving parts (including
  Salt or Puppet servers).

## What about Puppet?

To be honest, Puppet adapter needs some love. Formulas are not supported and
there's no plan to do so in a near future. What's more: it hasn't been properly
tested and I have to find out whether it is still working after recent changes.

Of course I'll find some time to invest on it but having some extra help would
be great. So, if you're interested, just ping me!

## Conclusion

YaST2 CM is still a young project and there's a lot of work to do. I would like
to continue adding new features and fixing bugs. And, of course, getting some
feedback would be really great!

So if you're interested in contributing (testing, documenting, writing code, etc.)
just let me know.
