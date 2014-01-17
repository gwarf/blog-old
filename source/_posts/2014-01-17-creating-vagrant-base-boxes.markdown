---
layout: post
title: ""Creating Vagrant Base boxes""
date: 2014-01-17 10:34:56 +0100
comments: true
categories: vagrant box sl
---

## Create a VM inside VirtualBox

* No usb
* No audio
* One dynamic disk (10GiB)
* 512 MiB of RAM
* One vcpu
* One network card in NAT mode

## Install the base system, as minimal as possible

### Scientific Linux 5
* Retrieve ISOs CD 1 and 2 from:
* Boot CD
* Use basic video driver installation
* Use default configuration
  * except:
    * DHCP for ipv6
    * Deselct every packages sets
* Use vagrant as root password

### Scientific Linux 6
* Retrieve netinstall ISO CD from:
* Boot CD
* Use network install URL:
* Use default configuration
* Use vagrant as root password

Debian 7: retrieve netinstall ISO CD from:

## System configuration

### Scientific Linux 5/6
* Update system and reboot if kernel was updated

``` sh
yum clean all && yum update -y
reboot
```

* Add a vagrant user with vagrant as password

``` sh
adduser vagrant
passd vagrant
```

* Configure password-less sudo for vagrant user

``` sh
visudo
Default:vagrant !requiretty
vagrant ALL=(ALL) NOPASSWD: ALL
```

* Configure ssh server

``` sh
sed -i 's/^#UseDNS yes/UseDNS no/' /etc/ssh/sshd_config
```

* Configure ssh authorized_keys for vagrant user

``` sh
mkdir ~vagrant/.ssh
curl -o ~/vagrant/.ssh/authorized_keys \
  https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub
chmod 0700 ~vagrant/.ssh
chmod 0600 ~vagrant/.ssh/authorized_keys
chown -R vagrant:vagrant ~vagrant/.ssh
```

### Scientific Linux 6

Minimal SL6 install does not install acpid nor perl

``` sh
yum install -y acpid perl
service acpid start
```

## VirtualBox Additions installation

### Scientific Linux 5/6

* Insert Guest additions CD image using VirtualBox device menu
* Install required software for build the VirtualBox additions

``` sh
yum install -y gcc makekernel-devel
```

* Build and install VirtualBox additions

Error about OpenGL or Window System drivers are "normal"

``` sh
mount /dev/cdrom /mnt
sh /mnt/VBoxLinuxAdditions.run
umount /mnt
```

## Cleaning image

### Scientific Linux 5/6

``` sh
yum clean all
: > /var/log/messages
: > /var/log/secure
: > ~/.bash_history
kill -9 $$
```

ACPI shutdown VM using VirtualBox Machine menu

## Packing the box

### Scientific Linux 5

``` sh
vagrant package --output sl5-64-VB436-nocm.box --base scientificlinux5
```

### Scientific Linux 6

``` sh
vagrant package --output sl6-64-VB436-nocm.box --base scientificlinux6
```
