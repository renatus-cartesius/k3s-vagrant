# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile for deploying k3s cluster

slave_hostname_prefix = "local-s"
master_hostname = "local-master"
slaves_count = 3

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/focal64"

  config.vm.define "#{master_hostname}" do |node|
    node.vm.env.NODE_TYPE = "master"
    node.vm.hostname = master_hostname
      # Adding SSH keys from host

    node.vm.provision "shell" do |s|
      ssh_pub_key = File.readlines("#{Dir.home}/.ssh/ansible_rsa.pub").first.strip
      s.inline = <<-SHELL
        echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
        echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
      SHELL
    end

    # Configuring network

    node.vm.network "public_network"

    # Setting main params

    node.vm.provider "virtualbox" do |vb|
      # Don`t display the VirtualBox GUI when booting the machine
      vb.gui = false

      # Customize the amount of memory on the VM:
      vb.memory = "2048"
      vb.cpus = 4
      vb.name = master_hostname
    end

  # Initialize packages and setting up repos
    node.vm.provision "ansible" do |ansible|
      ansible.become = true
      ansible.become_user = "root"
      ansible.verbose = "v"
      ansible.playbook = "ansible/deploy-cluster.yml"
    end
  end

  (1..slaves_count).each do |i|
    slave_hostname = "#{slave_hostname_prefix}#{i}"
    config.vm.define "#{slave_hostname}" do |node|
      node.vm.hostname = slave_hostname

      node.vm.env.NODE_TYPE = "slave"

      # Adding SSH keys from host

      node.vm.provision "shell" do |s|
        ssh_pub_key = File.readlines("#{Dir.home}/.ssh/ansible_rsa.pub").first.strip
        s.inline = <<-SHELL
          echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
          echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
        SHELL
      end

      # Configuring network

      node.vm.network "public_network"

      # Setting main params

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

end
