# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

conf = YAML.load_file 'conf.yml'
bootstrap = conf['bootstrap']
ntp = conf['ntp_server']
storage_node_count = conf['storage_node_count']
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

  (0..storage_node_count).each do |i|
    config.vm.define "node#{i}" do |hostconfig|
      hostconfig.vm.hostname = "node#{i}"
      hostconfig.vm.network "private_network", type: "dhcp"
      hostconfig.vm.provider "virtualbox" do |vb|
        unless File.exist?("node#{i}.vdi")
          vb.customize ['createhd', '--filename', "node#{i}", '--size', 1 * 1024]
        end
        vb.customize ['storageattach', :id, '--storagectl', "IDE Controller", '--port', "1", '--device', "1", '--type', 'hdd', '--medium', "node#{i}.vdi"]
        vb.name = "node#{i}"
      end

      if i == storage_node_count
        hostconfig.vm.provision :ansible do |ansible|
          ansible.limit = 'all'
          ansible.playbook = "network.yml"
        end

        hostconfig.vm.provision :ansible do |ansible|
          ansible.limit = 'all'
          ansible.groups = {
            'gluster_servers' => ["node[1:#{storage_node_count}]"],
          }
          ansible.playbook = 'filesystem.yml'
        end

        volume_string = "gluster volume create vol1 "
        for j in 1..storage_node_count do
          volume_string << "node#{j}:/bricks/brick1 "
        end
        volume_string << "force"

        hostconfig.vm.provision :ansible do |ansible|
          ansible.limit = 'all'
          ansible.groups = {
            'node1' => ["node1"],
            'other_storage_nodes' => ["node[2:#{storage_node_count}]"]
          }
          ansible.extra_vars = {
            "volume_string": volume_string
          }
          ansible.playbook = 'cluster.yml'
        end
      end

    end
  end
end
