---
layout: post
title: "Node-less puppet setup using hiera"
date: 2013-12-11 22:15
comments: true
categories: puppet hiera sysadmin
---
## Why?

Following a big puppet 2.7 => 3.3 space jump (it took quite some times
to test/setup/adapt everything) I am trying to get a cleaner/saner
puppet usage to avoid errors, duplication (allowing to easily override
some conf for a specific deployment site at some specific location) and
to avoid slapping my lazyness with a truit.

So I crawled a bit the web, and read a lot of different
posts/bugs/idas/rants, and did not find the golden-wonderfull-definitive
set-up guide, so here are the things that are on the way:
* Use hiera for storing the nodes configuration
* Assign classes using hiera (node-less setup?)
* Create roles and profiles modules to allow to encapsulate contents not
  configurable using hiera

## How?

### Hiera base setup

Nothing to fancy here as shown in the hiera.yaml file, just an
environment-dependent datadir and a first draft of the hierarchy that
will be used.

``` yaml hiera.yaml
---
:backends:
  - json
:json:
  :datadir: /etc/puppet/environments/%{::environment}/hieradata/
:hierarchy:
  - "%{::fqdn}"
  - "%{::company_role}"
  - "%{::company_location}"
  - "%{::virtual}"
  - "%{::operatingsystem}-${lsbdistrelease}"
  - "%{::operatingsystem}-${lsbmajdistrelease}"
  - "%{::operatingsystem}"
  - "%{::osfamily}"
  - common

# vim: set ft=yaml et smarttab sw=2 ts=2 sts=2:
```

### Custom facts for hiera

Here two hiera data sources are meant to be able to easily configure a
node according to its location or role. (location meaning more or
less a more or less physical location with some specific network
configuration or other specific rules/requirements)

In order to be able to assign the role and location, custom facts were
added (company_role and company_location), based on the content of a
file that have to be available on the server. (see XXX for more)

### Assigning class to nodes using hiera

``` json hieradata/common.js
{
  /* Load default classes */
  "classes" : [
    "unix",
    "skel",
    "requiredsoftware",
    "pamldap",
    "git",
    "liquidprompt",
    "ruby",
    "ruby::dev",
    "postfix",
    "ntp",
    "sudo",
  ],
```

``` ruby site.pp
# Load classes from hiera conf mergeing all classes for inclusion
hiera_include('classes')
```

### Assigning defines to nodes using hiera

``` json hieradata/common.js
  "rsyslog_configs" : {
    "iptables.conf" : {  "ensure" : "present", "source" : "puppet:///modules/site/rsyslog/rsyslog.d/iptables.conf" },
    "puppet-agent.conf" : {  "ensure" : "present", "source" : "puppet:///modules/site/rsyslog/rsyslog.d/puppet-agent.conf" },
  },
```

```
node default {
  # Retrieve rsyslog configurations
  $rsyslog_configs = hiera_hash('rsyslog_configs', {})
  create_resources('rsyslog::config', $rsyslog_configs)
}
```
