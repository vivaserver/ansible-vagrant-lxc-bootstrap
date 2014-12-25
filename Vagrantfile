# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.define "project" do |project|  # namespace to have a sensible container name
    project.vm.box = "fgrehm/precise64-lxc"
    project.vm.provider :lxc do |lxc|
      # lxc 0.7.5 on Ubuntu 12.04 "fix"
      # ref. https://github.com/fgrehm/vagrant-lxc#backingstore-options
      lxc.backingstore = 'none'
    end

    # ref. http://docs.vagrantup.com/v2/provisioning/ansible.html
    project.vm.provision :ansible do |ansible|
      ansible.playbook = "provisioning/project.yml"
      ansible.verbose = 'vv'
    end
  end

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network :forwarded_port, guest: 80, host: 8080

  # Warning! The LXC provider doesn't support any of the Vagrant public / private
  # network configurations (ex: `config.vm.network :private_network, ip: "some-ip"`).
  # config.vm.network :private_network, ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network :public_network

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  # config.ssh.forward_agent = true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"
end
