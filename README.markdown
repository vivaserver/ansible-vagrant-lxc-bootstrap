# Vagrant with LXC provider and Ansible provisioning bootstrap

Have just one Vagrantfile w/multiple VM configurations and deployment playbooks. Effectively documenting all the project's infrastructure.

refs.
  - http://pfigue.github.io/blog/2014/01/25/using-vagrant-with-lxc-linux-containers/
  - http://www.capsunlock.net/2014/02/vagrant-ansible-provisioning-add-roles-using-ansible-galaxy.html
  - https://github.com/cocoy/vagrant_ansible/blob/master/Vagrantfile


## vagrant-lxc plugin for Vagrant 1.6

Vagrant 1.6 allows the downloading of VM images from Vagrant cloud. See available LXC "boxes":

https://vagrantcloud.com/search?utf8=%E2%9C%93&sort=&provider=&q=lxc

And in your Vagrantfile:

    ...
    config.vm.box = "fgrehm/precise64-lxc"
    ...

Installation:

    $ vagrant plugin install vagrant-lxc --plugin-version 1.0.0.alpha.3

ref. https://github.com/fgrehm/vagrant-lxc#installation

Also note lxc 0.7.5 "fix" for Ubuntu 12.04 on your Vagrantfile:

    ...
    config.vm.provider :lxc do |lxc|
      lxc.backingstore = 'none'
    end
    ...

ref. https://github.com/fgrehm/vagrant-lxc#backingstore-options


## Provisioning VMs selectively

Bring up & provision the "lemp" box:

    $ vagrant up lemp --provider=lxc
    ...
    $ vagrant status lemp
    $ vagrant provision lemp
    ...
    $ vagrant halt lemp
    $ vagrant destroy lemp

Bring up & provision the "services" box:

    $ vagrant up services --provider=lxc
    ...
    $ vagrant status services
    $ vagrant provision services
    ...
    $ vagrant halt services
    $ vagrant destroy services

Previous single-Vagrantfile projects Git-tagged as follows:

    lamp            single Vagrantfile for lamp playbook
    mssql           single Vagrantfile for services playbook
    nginx           single Vagrantfile for lemp playbook


## Deploy a provisioning playbook to production 

(mind `- hosts: all` line on playbook):

    $ ansible-playbook --limit=mebac-jujuy.dyndns.org --inventory=provisioning/hosts -vv -K --sudo --tags=dyndns provisioning/services_backend.yml 

but first:

  * check for production values on each playbook
  * copy your ssh keys to new VPS host [1]
  * enable sudo privileges to your deployment user [2]

[1]: http://procbits.com/2013/09/08/getting-started-with-ansible-digital-ocean
[2]: https://www.digitalocean.com/community/articles/how-to-add-and-delete-users-on-ubuntu-12-04-and-centos-6


Remotely execute arbitrary command using local hosts file:

    $ ansible all -i provisioning/hosts -vv -K --sudo -m apt -a "upgrade=yes"  # Ansible 1.6
    ...
    $ ansible web -i provisioning/hosts -vv --ask-pass --ask-sudo-pass -m mysql_user -a "name=vivab0rg password= priv=*.*:ALL"
    ...
    $ ansible web -i provisioning/hosts -vv --ask-pass --ask-sudo-pass -a "reboot" -f 0
    ...
    $ ansible services -i provisioning/hosts -vv -u vagrant --sudo --private-key ~/.vagrant.d/insecure_private_key -m setup
    $ ansible-playbook --limit=lemp --inventory=provisioning/hosts -vv -u vagrant --sudo --private-key ~/.vagrant.d/insecure_private_key --tags=fpm-fix provisioning/site.yml


## Ansible

Execute arbitrary command on Vagrant machine:

    $ ansible 10.0.3.89 -vvv -u vagrant --sudo --private-key ~/.vagrant.d/insecure_private_key -m apt -a "upgrade=yes"

Target OS distributions on tasks:

    - name: ensure Quantal main repo.
      apt_repository: repo='deb http://archive.ubuntu.com/ubuntu quantal main'
      when: ansible_lsb.codename == 'precise'
      sudo: yes

More useful variables for "when":

    "ansible_architecture": "x86_64"
    "ansible_distribution": "Ubuntu",
    "ansible_distribution_release": "precise",
    "ansible_distribution_version": "12.04",

These need the `lsb_release` package:

    "ansible_lsb": {
      "codename": "precise",
      "description": "Ubuntu 12.04.2 LTS",
      "id": "Ubuntu",
      "major_release": "12",
      "release": "12.04"
    },

ref. http://docs.ansible.com/playbooks_variables.html

As a reminder, to see what facts are available on a particular system, you can do:

    $ ansible default -i provisioning/hosts -vv -u vagrant --sudo --private-key ~/.vagrant.d/insecure_private_key -m setup
    ...
    $ ansible localhost -vv -m setup

(c)2014 Cristian R. Arroyo
