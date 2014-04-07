---
layout: post
title: "Dependencies with Hiera and create_resources."
date: 2014-04-07 21:32:28 +0200
comments: true
categories: 
---
In a [nodeless setup](/blog/2013/12/11/node-less-puppet-setup-using-hiera) it
is possible to manage dependencies between resources created by
create_resources, but the syntax is quite strict and caused me some
troubles.
If the syntax is not correct the traditionnal ```Could not find
dependency``` error message will be displayed.

The following won't work:

``` ruby common.yaml
services:
  mysql:
    ensure: 'running'
    require: Package['mysql-server']
```

Nor the following:

``` ruby common.yaml
services:
  mysql:
    ensure: 'running'
    require: "Package['mysql-server']"
```

But the following two syntaxes will work:

``` ruby common.yaml
---
classes:
  - 'puppet::agent'
packages:
  mysql-server:
    ensure: 'installed'
services:
  mysql:
    ensure: 'running'
    require: Package[mysql-server]
```

``` ruby common.yaml
---
classes:
  - 'puppet::agent'
packages:
  mysql-server:
    ensure: 'installed'
services:
  mysql:
    ensure: 'running'
    require: 'Package[mysql-server]'
```

See a full example running in Vagrant at https://github.com/gwarf/puppet-vagrant-playground
