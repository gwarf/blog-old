---
layout: post
title: ""Puppet tests using Vagrant""
date: 2013-12-10 13:45
comments: true
categories: puppet sysadmin vagrant
---

XXX This is a work in progress.

## Intro

Adapted from:
* http://www.example42.com/?q=Example42%20Puppet%20Playground
* http://grahamgilbert.com/blog/2013/02/13/building-a-test-puppet-master-with-vagrant/
* http://theholyjava.wordpress.com/2013/09/03/test-puppet-config-of-an-existing-node-using-puppet-master-inside-vagrant/
* http://journal.ryanmccue.info/209/vagrant-puppet-master-pt-1/
* http://journal.ryanmccue.info/209/vagrant-puppet-master-pt-1/
* http://brokenhaze.com/blog/2013/07/25/puppet-workflow-with-vagrant/
* http://stackoverflow.com/questions/14168588/vagrant-running-a-puppet-master-with-a-puppet-agent
* http://java.dzone.com/articles/test-puppet-config-existing
* https://github.com/garethr/puppetmaster-vagrant

## Prerequisites

* [Vagrant](http://www.vagrantup.com/)
* [VirtualBox](https://www.virtualbox.org/)
* [git](http://git-scm.com/)

## Planned setup

* One puppet master running Debian 7 64bits
* One puppet client running Debian 7 64bits
* One puppet client running Scientific Linux 5.x 64 bits
* One puppet client running Scientific Linux 6.x 64 bits
* One puppet client running CentOS 6.x 64 bits

## Planned workflow

## Boxes URLs

Boxes:
* http://www.vagrantbox.es/
* http://puppet-vagrant-boxes.puppetlabs.com/

* Debian 7 64 bits - http://dl.dropboxusercontent.com/s/xymcvez85i29lym/vagrant-debian-wheezy64.box Scientific Linux 5.x 64 bits
* Scientific Linux 5.x 64 bits -
* Scientific Linux 6.x 64 bits -  http://lyte.id.au/vagrant/sl6-64-lyte.box
* CentOS 6.x 64 bits - http://puppet-vagrant-boxes.puppetlabs.com/centos-64-x64-vbox4210.box

## Box installation
Box installation will take some time as the boxes have to be downloaded locally.

```
vagrant box add debian7-64 http://dl.dropboxusercontent.com/s/xymcvez85i29lym/vagrant-debian-wheezy64.box
vagrant box add sl5-64 http://lyte.id.au/vagrant/sl6-64-lyte.box
vagrant box add centos6-64 http://lyte.id.au/vagrant/sl6-64-lyte.box
```

## Vagrant configuration
``` ruby Vagrantfile
# Vagrantfile which sets up one puppet master and one puppet client.
# Assumes "puppet-dev" and "scripts" repos are cloned into the same
# base directory.

Vagrant.configure("2") do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "debian7-64"

  # Setup the Puppet master
  config.vm.define :master do |master|
    master.vm.box = "debian7-64"
    master.vm.box_url = "http://dl.dropboxusercontent.com/s/xymcvez85i29lym/vagrant-debian-wheezy64.box"
    master.vm.hostname = "puppet.local"
    #master.vm.synced_folder "../../puppet-dev", "/etc/puppet"
    #master.vm.synced_folder "../../scripts", "/root/scripts"
    master.vm.network :private_network, ip: "192.168.1.42"
    #master.vm.provision :shell, :path => "master.sh"
    # Customize the actual virtual machine
    master.vm.provider :virtualbox do |vb|
      # Uncomment this, and adjust as needed to add memory to vm
      vb.customize ["modifyvm", :id, "--memory", 2048]
      # Because Virtual box is stupid - change the default nat network
      vb.customize ["modifyvm", :id, "--natnet1", "192.168.1.0/24"]
    end
  end

  # Setup the Puppet client. You can copy and modify this stanza to allow for
  # multiple client, just change all instances of 'client1' to another term
  # such as 'client2'
  config.vm.define :client1 do |client1|
    client1.vm.box = "debian7-64"
    client1.vm.box_url = "http://dl.dropboxusercontent.com/s/xymcvez85i29lym/vagrant-debian-wheezy64.box"
    client1.vm.hostname = "client1.local"
    # Make puppet-dev accessable from the client for easier copying.
    #client1.vm.synced_folder "../../puppet-dev", "/root/puppet"
    client1.vm.network :private_network, ip: "192.168.1.43"
    #client1.vm.network :forwarded_port, guest: 8100, host: 8100
    #client1.vm.provision :shell, :path => "client.sh"
    client1.vm.provider :virtualbox do |vb|
      # Uncomment this, and adjust as needed to add memory to vm
      vb.customize ["modifyvm", :id, "--memory", 1024]
      # Because Virtual box is stupid - change the default nat network
      vb.customize ["modifyvm", :id, "--natnet1", "192.168.1.0/24"]
    end
  end

end
```

``` ruby
Vagrant.configure("2") do |config|
  config.cache.auto_detect = true
  # ...
end
Vagrant::Config.run do |config|
  {
    :Centos6_64 => {
      :box => 'centos6_64',
      :box_url => 'https://dl.dropbox.com/u/7225008/Vagrant/CentOS-6.3-x86_64-minimal.box',
    },
    :Debian7_64 => {
      :box => 'wheezy64',
      :box_url => 'https://dl.dropboxusercontent.com/u/86066173/debian-wheezy.box',
    },
  }.each do |name,cfg|
    config.vm.define name do |local|
      local.vm.box = cfg[:box]
      local.vm.box_url = cfg[:box_url]
# local.vm.boot_mode = :gui
      local.vm.host_name = ENV['VAGRANT_HOSTNAME'] || name.to_s.downcase.gsub(/_/, '-').concat(".gnubila.fr")
      local.vm.provision :puppet do |puppet|
        puppet.manifests_path = "manifests"
        puppet.module_path = "modules"
        puppet.manifest_file = "init.pp"
        puppet.options = [
         '--verbose',
         '--report',
         '--show_diff',
# '--debug',
# '--parser future',
        ]
      end
    end
  end
end
```

## Later
* Creating custom boxes using veewee
https://github.com/jedi4ever/veewee
https://github.com/puppetlabs/puppet-vagrant-boxes
