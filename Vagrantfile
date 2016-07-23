# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.define "smgr" do |smgr|
	smgr.vm.box = "ubuntu/trusty64"
	smgr.vm.hostname = "smgr"
	smgr.vm.network "private_network", ip: "192.168.33.10"
  end
  
  config.vm.define "snode1" do |snode1|
	snode1.vm.box = "ubuntu/trusty64"
	snode1.vm.hostname = "snode1"
	snode1.vm.network "private_network", ip: "192.168.33.20"
  end
  
  config.vm.define "snode2" do |snode2|
	snode2.vm.box = "ubuntu/trusty64"
	snode2.vm.network "private_network", ip: "192.168.33.30"
	snode2.vm.hostname = "snode2"
  end

  config.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
  end
end
