# -*- mode: ruby -*-
# vi: set ft=ruby :

master_count = (ENV["K8S_MASTERS"] || "3").to_i
worker_count = (ENV["K8S_WORKERS"] || "3").to_i
vm_memory = (ENV["K8S_VM_MEMORY"] || "1024").to_i
vm_image = ENV["K8S_VM_IMAGE"] || "bento/ubuntu-18.04" # "bento/centos-7"

Vagrant.configure("2") do |config|

  config.vm.box = vm_image
  config.vm.box_check_update = false

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 1
    vb.memory = vm_memory
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end
  
  (0..(master_count - 1)).each do |i|
    config.vm.define "controller-#{i}" do |node|
      node.vm.hostname = "controller-#{i}"
      node.vm.network "private_network", ip: "192.168.100.10#{i}"
      node.vm.provider "virtualbox" do |vb|
        vb.name = "controller-#{i}"
      end
    end
  end
  
  (0..(worker_count - 1)).each do |i|
    config.vm.define "worker-#{i}" do |node|
      node.vm.hostname = "worker-#{i}"
      node.vm.network "private_network", ip: "192.168.100.20#{i}"
      node.vm.provider "virtualbox" do |vb|
        vb.name = "worker-#{i}"
      end
    end
  end

end
