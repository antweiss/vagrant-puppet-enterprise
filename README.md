# vagrant-puppet-enterprise

This Vagrantfile will setup a Multi-VM vagrant environment based on Puppet Enterprise with a Puppet Master and one or more Puppet Agents. Using this setup, you can develop and test Puppet modules that have more than trivial functionality (e.g. exported resources, etc).

Note: Puppet Enterprise is free to use up to 10 nodes. If you use more than 10 nodes, please [buy the appropriate licences](https://puppetlabs.com/puppet/how-to-buy/).

## Setup

Specifications:
* Centos 6.4 x86_64 minimal
* Puppet Enterprise 2.8.0
* Put manifests in: `manifests/` folder
* Put modules in: `modules/` folder
* Puppet dashboard: https://192.168.111.111 (login: puppet@example.com / puppet@example.com)

Requirements:
* VirtualBox 4.2
* Vagrant 1.1

The setup is based on multi-machine vagrant setup; there is 1 puppet master, and 1 or more agents. Both types have their own basebox:
* Master: https://s3-eu-west-1.amazonaws.com/xebia-vm/vagrant-boxes/centos-64-x86_64-minimal-pe-master.box
* Agent: https://s3-eu-west-1.amazonaws.com/xebia-vm/vagrant-boxes/centos-64-x86_64-minimal-pe-agent.box

The puppet master has a fixed IP address: 192.168.111.111. You can add as many agents as you like, but be sure to give them a unique IP address in the 192.168.111.x range (.1 and .111 are taken). To do so, duplicate the agent block in the `Vagrantfile` and `site.pp` files.

Configuration details for the puppet master:
* Will autosign agent certificates
* Storeconfigs is enabled
* Fileserver is enabled, files served from local files folder (use `puppet:///files/<file_to_be_served>` in puppet plans), used fileserver.conf:

```
[files]
path /etc/puppetlabs/puppet/files
allow *
```

* Hiera enabled, put hiera files in hieradata folder, used hiera.yaml config:

```
---
:backends:
  - yaml
:hierarchy:
  - %{::clientcert}
  - %{::environment}
  - common
:yaml:
  :datadir: /etc/puppetlabs/puppet/hieradata
```

## Templates

Use the Vagrantfile and site.pp templates below and replace the following fields:
* `<manifests_local_path>` - Local path for the puppet manifests. Must contain the site.pp file (see template below).
* `<modules_local_path>` - Local path for the puppet modules. Will probably point to a local git repository
* `<files_local_path>` - Local path for the puppet fileserving.
* `<hieradata_local_path>` - Local path for the hiera data files.
* `<agent_name>` - Name for the agent. Used within Vagrant (e.g. `vagrant provision agent_name`)
* `<agent_hostname>` - Hostname for the agent. This links vagrant with puppet. Must be unique within this Vagrantfile.
* `<agent_ipaddress>` - IP adress for the agent. Must be unique within all running VirtualBox images.

### Vagrantfile template

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.define :master do |master|
    master.vm.hostname = "puppet"
    master.vm.box = "centos-64-x86_64-minimal-pe-master"
    master.vm.box_url = "https://s3-eu-west-1.amazonaws.com/xebia-vm/vagrant-boxes/centos-64-x86_64-minimal-pe-master.box"

    master.vm.network :private_network, ip: "192.168.111.111"
    master.vm.network :forwarded_port, guest: 443, host: 8443

    master.vm.synced_folder "<manifests_local_path>", "/etc/puppetlabs/puppet/manifests"
    master.vm.synced_folder "<modules_local_path>", "/etc/puppetlabs/puppet/modules"
    master.vm.synced_folder "<files_local_path>", "/etc/puppetlabs/puppet/files"
    master.vm.synced_folder "<hieradata_local_path>", "/etc/puppetlabs/puppet/hieradata"
  end

  config.vm.define :<agent_name> do |agent|
    agent.vm.hostname = "<agent_hostname>"
    agent.vm.box = "centos-64-x86_64-minimal-pe-agent"
    agent.vm.box_url = "https://s3-eu-west-1.amazonaws.com/xebia-vm/vagrant-boxes/centos-64-x86_64-minimal-pe-agent.box"

    agent.vm.network :private_network, ip: "<agent_ipaddress>"

    agent.vm.provision :puppet_server do |agent_puppet|
      agent_puppet.options = "--test --waitforcert"
    end
  end
end
```

### site.pp template

```
# Make filebucket 'main' the default backup location for all File resources:
filebucket { 'main':
  server => 'puppet',
  path => false,
}
File { backup => 'main' }

node default {}

node '<agent_hostname>' {
  ... resource declarations ...
}
```

## Caveats

When destroying and recreating an agent while keeping the puppet master image, you will get a certificate error:

	err: Could not request certificate: The certificate retrieved from the master does not match the agent's private key.

This is because the agent will generate a new certificate when the box is recreated. The old certificate is still known with the puppet master. Two options:
* Destroy and recreate the master as well
* Revoke the old agent certificate on the puppet master and remove the certificate from the agent:

```
vagrant ssh master -c 'sudo puppet cert clean <agent_hostname>'
vagrant ssh agent -c 'sudo rm /etc/puppetlabs/puppet/ssl/certs/<agent_hostname>.pem'
```
