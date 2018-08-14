# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

conf = YAML.load_file 'conf.yml'
bootstrap = conf['bootstrap']
ntp = conf['ntp_server']
username = conf['rhel_username']
password = conf['rhel_password']

Vagrant.configure(2) do |config|
  config.vm.box = "generic/rhel7"
  config.ssh.forward_x11 = true
  config.ssh.insert_key = false

  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.cpus = 2
    vb.memory = 2048
  end
 
  config.vm.provision :shell, :path => bootstrap, :args => [ntp, username, password]

  (0..3).each do |node_index|
    config.vm.define "node#{node_index}" do |machine|
      machine.vm.hostname = "node#{node_index}"
      machine.vm.network "private_network", type: "dhcp"
      machine.vm.provider "virtualbox" do |vb|
        unless File.exist?("node#{node_index}.vdi")
          vb.customize ['createhd', '--filename', "node#{node_index}", '--size', 1 * 1024]
        end
        vb.customize ['storageattach', :id, '--storagectl', "IDE Controller", '--port', "1", '--device', "1", '--type', 'hdd', '--medium', "node#{node_index}.vdi"]
        vb.name = "node#{node_index}"
      end

      if node_index == 3
        
        machine.vm.provision :ansible do |ansible|
          ansible.limit = 'all'
          ansible.playbook = "network.yml"
        end

        machine.vm.provision :ansible do |ansible|
          ansible.limit = 'all'
          ansible.groups = {
            'gluster_servers' => ["node[1:3]"],
          }
          ansible.playbook = 'filesystem.yml'
        end

        machine.vm.provision :ansible do |ansible|
          ansible.limit = 'all'
          ansible.groups = {
            'node1' => ["node1"],
          }
          ansible.playbook = 'cluster.yml'
        end

      end

    end
  end
end
