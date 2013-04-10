# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define :master do |master|
    master.vm.hostname = "puppet"
    master.vm.box      = "vagrant-centos-64-x86_64-pe-master"

    master.vm.network :private_network, ip: "192.168.111.111"
    master.vm.network :forwarded_port, guest: 443, host: 8443

    master.vm.synced_folder "manifests", "/etc/puppetlabs/puppet/manifests"
    master.vm.synced_folder "modules", "/etc/puppetlabs/puppet/modules"
  end

  config.vm.define :agent do |agent|
    agent.vm.hostname = "agent"
    agent.vm.box      = "vagrant-centos-64-x86_64-pe-agent"

    agent.vm.network :private_network, ip: "192.168.111.222"

    agent.vm.provision :puppet_server do |agent_puppet|
      agent_puppet.options = "--test --waitforcert"
    end
  end
end
