# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile for deploying k3s cluster

ENV['VAGRANT_EXPERIMENTAL'] = "1"

slave_hostname_prefix = "local-s"
master_hostname = "local-master"
slaves_count = 3

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/focal64"
  config.ssh.insert_key = false
  config.ssh.private_key_path = ["#{Dir.home}/.ssh/ansible_rsa", "#{Dir.home}/.vagrant.d/insecure_private_key"]
  
  config.vm.define "#{master_hostname}" do |node|
    # Setup node_type env for ansible provisioning
    node_type = "master"

    # Setup hostname
    node.vm.hostname = master_hostname
    

    # Adding SSH keys from host and setting up node_type file
    node.vm.provision "shell" do |s|
      ssh_pub_key = File.readlines("#{Dir.home}/.ssh/ansible_rsa.pub").first.strip
      s.inline = <<-SHELL
        echo #{ssh_pub_key} > /home/vagrant/.ssh/authorized_keys
        echo #{ssh_pub_key} > /root/.ssh/authorized_keys
        echo #{node_type} > /root/.nodetype
      SHELL
    end

    # Configuring network
    node.vm.network "public_network"

    # Setting up virtualbox params
    node.vm.provider "virtualbox" do |vb|
      # Don`t display the VirtualBox GUI when booting the machine
      vb.gui = false

      # Customize the amount of memory on the VM:
      vb.memory = "2048"
      vb.cpus = 4
      vb.name = master_hostname
    end

  end

  (1..slaves_count).each do |i|
    slave_hostname = "#{slave_hostname_prefix}#{i}"
    config.vm.define "#{slave_hostname}" do |node|

      # Setup node_type env for ansible provisioning
      node_type = "slave"

      node.vm.hostname = slave_hostname

      # Adding SSH keys from host and setting up node_type file
      node.vm.provision "shell" do |s|
        ssh_pub_key = File.readlines("#{Dir.home}/.ssh/ansible_rsa.pub").first.strip
        s.inline = <<-SHELL
          echo #{ssh_pub_key} > /home/vagrant/.ssh/authorized_keys
          echo #{ssh_pub_key} > /root/.ssh/authorized_keys
          echo #{node_type} > /root/.nodetype
        SHELL
      end

      # Configuring network
      node.vm.network "public_network"

      # Setting up virtualbox params
      node.vm.provider "virtualbox" do |vb|

        # Don`t display the VirtualBox GUI when booting the machine
        vb.gui = false

        # Customize the amount of memory on the VM:
        vb.memory = "2048"
        vb.cpus = 4
        vb.name = slave_hostname
      end
    end
  end

  # Run ansible provisioning
  config.vm.provision "ansible", after: :all do |ansible|
    ansible.verbose = "v"
    ansible.compatibility_mode = "2.0"
    ansible.playbook = "ansible/deploy-cluster.yml"
  end
end
