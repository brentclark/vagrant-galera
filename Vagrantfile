# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

def configure_server(cluster_addresses)
  setup = <<-SCRIPT
#echo 'Acquire::http { Proxy "http://10.0.0.114:3142"; };' > /etc/apt/apt.conf
export DEBIAN_FRONTEND=noninteractive

apt-get update
apt-get upgrade -y
apt-get install -y software-properties-common python-software-properties software-properties-common
apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xcbcb082a1bb943db
apt-key adv --keyserver keyserver.ubuntu.com --recv BC19DDBA
echo 'deb http://releases.galeracluster.com/debian wheezy main' > /etc/apt/sources.list.d/galera.list
add-apt-repository 'deb http://chs-mirror02-v.cpt4.chs.hetzner.co.za/debian/ wheezy main'
apt-get update
apt-get install -y rsync galera-3 galera-arbitrator-3 mysql-wsrep-5.6

echo "
[mysqld]
query_cache_size=0
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
query_cache_type=0
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration
wsrep_cluster_name="test_cluster"
wsrep_cluster_address="gcomm://#{cluster_addresses.join(',')}"

# Galera Synchronization Configuration
wsrep_sst_method=rsync
" > /etc/mysql/conf.d/cluster.cnf
  SCRIPT
end

def configure_node(address)
  node = <<-SCRIPT
echo "[mysqld]
# Galera Node Configuration
wsrep_node_address=#{address}" > /etc/mysql/conf.d/node.cnf
  SCRIPT
end

CLUSTER_NODES = {
  "Galera1" => "10.0.0.200",
  "Galera2" => "10.0.0.201",
  "Galera3" => "10.0.0.202"
}

Vagrant.configure("2") do |config|
  config.ssh.forward_agent = true
  config.vm.box            = "debian/wheezy64"

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "1024"]
  end

  config.vm.define "Galera1" do |g1|
    g1.vm.provision "shell", inline: configure_server(CLUSTER_NODES.values)
    g1.vm.provision "shell", inline: configure_node("10.0.0.200")
    g1.vm.provision "shell", inline: "service mysql stop"
    #g1.vm.network "public_network", :bridge => "br0", type: "dhcp"
    g1.vm.network "public_network", :bridge => "br0", ip: "10.0.0.200"
  end

  config.vm.define "Galera2" do |g2|
    g2.vm.provision "shell", inline: configure_server(CLUSTER_NODES.values)
    g2.vm.provision "shell", inline: configure_node("10.0.0.201")
    g2.vm.provision "shell", inline: "service mysql stop"
    g2.vm.network "public_network", :bridge => "br0", ip: "10.0.0.201"
  end

  config.vm.define "Galera3" do |g3|
    g3.vm.provision "shell", inline: configure_server(CLUSTER_NODES.values)
    g3.vm.provision "shell", inline: configure_node("10.0.0.202")
    g3.vm.provision "shell", inline: "service mysql stop"
    g3.vm.network "public_network", :bridge => "br0", ip: "10.0.0.202"
  end
end
