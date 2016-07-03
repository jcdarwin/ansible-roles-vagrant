# -*- mode: ruby -*-
# vi: set ft=ruby :

# As per https://www.simonholywell.com/post/2016/02/intelligent-vagrant-and-ansible-files/
require 'yaml'
settings = YAML.load_file 'vagrant.yml'

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = settings['box']

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL

  # Ensure we can access our guest without have to port forward https
  # You'll also need a line in /private/etc/hosts:
  # 192.168.33.66 local.whatever
  config.vm.network "public_network", ip: settings['ip_address']
  	#,
	#"forwarded_port", guest: 80, host: 8000   # http

  config.vm.provider :virtualbox do |v|
    v.name = settings['vm_name']

	# Comment the following line to make our box headless
	v.gui = true

	# As per https://www.simonholywell.com/post/2016/02/intelligent-vagrant-and-ansible-files/
    # taken from http://www.stefanwrobel.com/how-to-make-vagrant-performance-not-suck#toc_1
    # assigns all available CPU cores and 1/4 of the host systems memory to the vm
    host = RbConfig::CONFIG['host_os']

    # Give VM 1/4 system memory & access to all cpu cores on the host
    if host =~ /darwin/
      cpus = `sysctl -n hw.ncpu`.to_i
      # sysctl returns Bytes and we need to convert to MB
      mem = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 4
    elsif host =~ /linux/
      cpus = `nproc`.to_i
      # meminfo shows KB and we need to convert to MB
      mem = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 4
    else # sorry Windows folks, I can't help you
      cpus = 2
      mem = 1024
    end

    v.customize ["modifyvm", :id, "--memory", mem]
    v.customize ["modifyvm", :id, "--cpus", cpus]
  end

  # Ensure our vhost public folder is synched with the correct ownership and permissions
  config.vm.synced_folder settings['public_folder'], "/var/www/site/current/public",
    id: settings['vm_name'],
    owner: "www-data",
    group: "www-data",
    mount_options: ["dmode=775,fmode=664"]

  # Setup the ansible inventory file
  Dir.mkdir(settings['ansible_inventory_dir']) unless Dir.exist?(settings['ansible_inventory_dir'])
  File.open("#{settings['ansible_inventory_dir']}/hosts" ,'w') do |f|
    f.write "[#{settings['vm_name']}]\n"
    f.write "#127.0.0.1 ansible_ssh_port=2222 ansible_ssh_user=#{settings['ssh_user']} ansible_ssh_private_key_file=#{settings['private_key']}\n"
    f.write "#{settings['ip_address']} ansible_ssh_port=#{settings['ssh_port']} ansible_ssh_user=#{settings['ssh_user']} ansible_ssh_private_key_file=#{settings['private_key']}\n"
  end

end
