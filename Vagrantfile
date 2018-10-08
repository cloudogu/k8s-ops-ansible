# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  #config.vm.box = "bento/ubuntu-18.04"
  config.vm.box = "bento/centos-7"
  config.vm.box_check_update = false

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 1
    vb.memory = 1024
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end
  
  (0..2).each do |i|
    config.vm.define "controller-#{i}" do |node|
      node.vm.hostname = "controller-#{i}"
      node.vm.network "private_network", ip: "192.168.100.10#{i}"
      node.vm.provider "virtualbox" do |vb|
        vb.name = "controller-#{i}"
      end
    end
  end
  
  (0..2).each do |i|
    config.vm.define "worker-#{i}" do |node|
      node.vm.hostname = "worker-#{i}"
      node.vm.network "private_network", ip: "192.168.100.20#{i}"
      node.vm.provider "virtualbox" do |vb|
        vb.name = "worker-#{i}"
      end
    end
  end

end
