# vagrant-dsc-cassandra

Single node Cassandra 2.2.6 (Datastax Community Edition) with OpsCenter and other swag.

# Prerequisites

The only things you will need are
* [VirtualBox](https://www.virtualbox.org)
* [Vagrant](https://www.vagrantup.com)
* 3GB of free RAM or you decrease the VM RAM settings via `export CFG_MEMSIZE=2048`

# Install

To bring up the Vagrant box run

```
git clone https://github.com/afritzler/vagrant-dsc.git
cd vagrant-dsc
vagrant up
```

# Use

To access the machine

```
vagrant ssh
```

The OpsCenter WebUI can be accessed via http://10.10.10.10:8888

# Remove

From the project folder execute

```
vagrant destroy
```
