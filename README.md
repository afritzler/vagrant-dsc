# vagrant-dsc-cassandra

Single node Cassandra 2.2.6 (Datastax Community Edition) with OpsCenter and other swag.

# Prerequisites

The only thing you will need is [Vagrant](https://www.vagrantup.com).

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
