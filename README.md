# What is this?

This is a project for running up a Vagrant Ubuntu virtual machine that can then be provisioned using Ansible.


## Prerequisites

Install vagrant (presuming we're on Mac OSX and we've installed [homebrew](http://brew.sh/)):

    brew cask install vagrant
    brew cask install vagrant-manager

    # Install the plugin to allow us to use shared folders
    vagrant plugin install vagrant-vbguest

Install virtualbox:

    brew cask install virtualbox

    # Install the extensions for our version of virtualbox
    # http://alanthing.com/blog/2013/03/17/virtualbox-extension-pack-with-one-line/
    export version=$(/usr/bin/vboxmanage -v) && \
	export var1=$(echo ${version} | cut -d 'r' -f 1) && \
	export var2=$(echo ${version} | cut -d 'r' -f 2) && \
	export file="Oracle_VM_VirtualBox_Extension_Pack-${var1}-${var2}.vbox-extpack" && \
	curl --silent --location http://download.virtualbox.org/virtualbox/${var1}/${file} -o ~/Downloads/${file} && \
	VBoxManage extpack install ~/Downloads/${file} --replace && \
	rm ~/Downloads/${file} && \
	unset version var1 var2 file

We use an Ubuntu 14.04 (LTS) vagrant box obtained from https://github.com/kraksoft/vagrant-box-ubuntu

    vagrant box add ubuntu/trusty64 \
	    https://github.com/kraksoft/vagrant-box-ubuntu/releases/download/14.04/ubuntu-14.04-amd64.box


## Install our server using vagrant

Our important settings live in `vagrant.yml`.
This file acts as a shared configuration for Vagrant and Ansible as per [these notes](https://www.simonholywell.com/post/2016/02/intelligent-vagrant-and-ansible-files/).

Once we're happy with our `vagrant.yml` settings, start up our virtual machine:

    # Start our box
    vagrant up

    # check that we can access it
    vagrant ssh

Ensure we can ssh normally:

	# Write the ssh config to vagrant-ssh
    vagrant ssh-config > vagrant-ssh

	# Use the vagrant-ssh ssh config
    ssh -F vagrant-ssh default


### Check that ansible can communicate with our vagrant box

Install Ansible if you haven't already:

    sudo pip install ansible

Check that we can ping our host:

    ansible -m ping all -i ansible/hosts
    ansible all -m ping -i ansible/hosts -l whatever
    ansible -m shell -a 'free -m' whatever -i ansible/hosts

If we want, we can add our public key to the server:

    cat ~/.ssh/id_rsa.pub | ssh -F vagrant-ssh default "cat >> ~/.ssh/authorized_keys"


### Folder sharing

We ensure our vhost folder is shared with the correct ownership and permissions
by adding the following to our `Vagrantfile`:

    config.vm.synced_folder "site", "/var/www/site/current",
        id: "site",
        owner: "www-data",
        group: "www-data",
        mount_options: ["dmode=775,fmode=664"]


### Private network

*We can't get this working properly, so use port-forwarding instead*
Ensure we can access our guest without have to port forward https, by adding
the following to our `Vagrantfile`:

  config.vm.network :private_network, ip: "192.168.1.100"

We'll also need a line in `/private/etc/hosts`:

	echo "192.168.1.100 local.vagrant" >> /private/etc/hosts

The above will let us access our guest at http://local.vagrant


### Port-forwarding

If instead we want to use port forwarding instead of a private network:

    # Add the following rule to our `Vagrantfile` to forward http ports
    config.vm.network "forwarded_port", guest: 80, host: 8000   # http

    # Add an entry to our `/private/etc/hosts` files
	echo "127.0.0.1 local.vagrant" >> /private/etc/hosts
