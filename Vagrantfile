# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define :master do |master|
    master.vm.hostname = "puppet"
    master.vm.box      = "pe-master-3.7.2"
    master.vm.box_url  = "file:///Users/antweiss/vagrant/vagrant-puppet-enterprise/pe-master3.7.2"
#    master.vm.box_url  = "xxxxxhttps://s3-eu-west-1.amazonaws.com/xebia-vm/vagrant-boxes/centos-64-x86_64-minimal-pe-master.box"

    master.vm.network :private_network, ip: "192.168.111.111"
    master.vm.network :forwarded_port, guest: 443, host: 8443

    master.vm.provider "virtualbox" do |v|
 	 v.name = "pe-master"
    end

  #  master.vm.synced_folder "manifests", "/etc/puppetlabs/puppet/manifests"
  #  master.vm.synced_folder "modules", "/etc/puppetlabs/puppet/modules"
  #  master.vm.synced_folder "files", "/etc/puppetlabs/puppet/files"
  #  master.vm.synced_folder "hieradata", "/etc/puppetlabs/puppet/hieradata"
  end

  config.vm.define :agent do |agent|
    agent.vm.hostname = "agent"
    agent.vm.box      = "pe-agent"
    agent.vm.box_url  = "file:///Users/antweiss/vagrant/vagrant-puppet-enterprise/pe-agent"
    agent.vm.network :private_network, ip: "192.168.111.222"
    agent.vm.provider "virtualbox" do |v|
         v.name = "pe-agent"
    end

  end
end
