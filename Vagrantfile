# -*- mode: ruby -*-
# vi: set ft=ruby :

# Adjustable settings
CFG_MEMSIZE = "3000"    # max memory for each VM
CFG_TZ = "US/Pacific"   # timezone, like US/Pacific, US/Eastern, UTC, Europe/Warsaw, etc.
NETWORK = '10.10.10.'   # base IP for DSC nodes
FIRST_IP = 10

# number of nodes to create
DSC_NODES = 1

# if local Debian proxy configured (DEB_CACHE_HOST), install and configure the proxy client
deb_cache_cmds = ""
if ENV['DEB_CACHE_HOST']
  deb_cache_host = ENV['DEB_CACHE_HOST']
  deb_cache_cmds = <<CACHE
apt-get install squid-deb-proxy-client -y
echo 'Acquire::http::Proxy "#{deb_cache_host}";' | sudo tee /etc/apt/apt.conf.d/30autoproxy
echo "Acquire::http::Proxy { debian.datastax.com DIRECT; };" | sudo tee -a /etc/apt/apt.conf.d/30autoproxy
cat /etc/apt/apt.conf.d/30autoproxy
CACHE
end


# Provisioning script for DSC nodes (dsc0, dsc1, ...)
# todo: better dynamic generation of /etc/hosts list
node_script = <<SCRIPT
#!/bin/bash
cat > /etc/hosts <<EOF
127.0.0.1       localhost

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

10.10.10.10   dsc0
EOF

mkdir -p /etc/opscenter/clusters/
cat > /etc/opscenter/clusters/Test_Cluster.conf <<EOF
[jmx]
username =
password =
port = 7199

[agents]

[cassandra]
username =
seed_hosts = localhost
password =
cql_port = 9042
EOF

# set timezone
echo "#{CFG_TZ}" > /etc/timezone
dpkg-reconfigure -f noninteractive tzdata

#{deb_cache_cmds}

# configure DataStax repository
echo "deb http://debian.datastax.com/community stable main" | sudo tee -a /etc/apt/sources.list.d/datastax.sources.list
wget -q -O - http://debian.datastax.com/debian/repo_key | sudo apt-key add -
apt-get update

# install java
echo "java 8 installation"
apt-get install --yes python-software-properties
add-apt-repository ppa:webupd8team/java
apt-get update -qq
echo debconf shared/accepted-oracle-license-v1-1 select true | /usr/bin/debconf-set-selections
echo debconf shared/accepted-oracle-license-v1-1 seen true | /usr/bin/debconf-set-selections
apt-get install --yes oracle-java8-installer
yes "" | apt-get -f install


# install DSC, Java, and a few base packages
apt-get install vim curl zip unzip git python-pip datastax-agent opscenter dsc22 cassandra=2.2.6 -y

# configure DSC
echo "Configuring DataStax Enterprise"
sudo sed -i 's/- seeds:.*$/- seeds: dsc0/'             /etc/cassandra/cassandra.yaml
sudo sed -i 's/listen_address:.*$/listen_address: %s/' /etc/cassandra/cassandra.yaml
sudo sed -i 's/rpc_address:.*$/rpc_address: %s/'       /etc/cassandra/cassandra.yaml

# start DSC
echo "Starting DataStax Community Edition"
sudo service cassandra start
sudo service opscenterd start

echo "Vagrant provisioning complete"
SCRIPT

# Configure VM servers
puts "Configured for #{DSC_NODES} DSC node(s)"
servers = []
(0..DSC_NODES-1).each do |i|
  name = 'dsc' + i.to_s
  ip = NETWORK + (FIRST_IP + i).to_s
  servers << {'name' => name, 'ip' => ip}
end

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"
  servers.each do |server|
    config.vm.define server['name'] do |config2|
      config2.vm.hostname = server['name']
      server_ip = server['ip']
      config2.vm.network :private_network, ip: server_ip
      config2.vm.provision :shell, :inline => node_script % [server_ip, server_ip]

      config2.vm.provider "vmware_fusion" do |v|
        v.vmx["memsize"]  = CFG_MEMSIZE
      end
      config2.vm.provider :virtualbox do |v|
        v.name = server['name']
        v.customize ["modifyvm", :id, "--memory", CFG_MEMSIZE]
      end

    end
  end
end
