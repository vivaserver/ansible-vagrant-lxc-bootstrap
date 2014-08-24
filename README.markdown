# Vagrant with LXC provider and Ansible provisioning bootstrap

See nice introduction at http://pfigue.github.io/blog/2014/01/25/using-vagrant-with-lxc-linux-containers/

## Sample usage

Have a `provisioning/project.yaml` file in your Vagrant directory with the following contents:

    ---
    - hosts: all
      handlers:
      - include: handlers/lamp.yml
      tasks:
      - include: tasks/dotfiles.yml
      - include: tasks/lamp.yml
      - include: tasks/phpmyadmin.yml

Now include it in your `Vagrantfile` as a provider:

    config.vm.provision :ansible do |ansible|
      ansible.playbook = "provisioning/project.yml"
      ansible.verbose = 'vv'
    end


## Vagrant

Tested against these package versions:

- Vagrant 1.3.5
- vagrant-lxc plugin 0.7.0
- Ansible 1.5

### Boxes

Initially based off stock `vagrant-lxc-precise-amd64-2013-09-28.box` image.

#### Custom LXC-enabled box

Vagrant box available at `vagrant-lxc-precise64-base-21.10.13.box`, generally imported as "lxc-precise64-base".

* Chef-related packages removed
* Ansible-related apt-enabling package added
* Base packages updated as of 21.10.13

#### Repackaging

Repack the base VM box from current using:

    $ vagrant package

ref: https://github.com/fgrehm/vagrant-lxc/issues/195


## Ansible

Remotely execute arbitrary command on Vagrant machine:

    $ cd provisioning
    $ ansible-playbook prep.yml --limit=prep --tags=ruby --inventory=../vagrant_ansible_inventory_prep -vv -u vagrant --sudo --private-key ~/.vagrant.d/insecure_private_key
    ...
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

(c)2014 Cristian R. Arroyo
