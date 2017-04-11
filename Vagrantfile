# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  ###############################################################################
  # Base box                                                                    #
  ###############################################################################
  config.vm.box = "centos/7"

  ###############################################################################
  # SSH Settings                                                                #
  ###############################################################################
  config.ssh.insert_key = false

  ###############################################################################
  # Global plugin settings                                                      #
  ###############################################################################
  plugins = ["vagrant-hostmanager"]
  plugins.each do |plugin|
    unless Vagrant.has_plugin?(plugin)
      raise plugin << " has not been installed."
    end
  end

  # set to false, if you do NOT want to check the correct VirtualBox Guest Additions version when booting this box
  if defined?(VagrantVbguest::Middleware)
    config.vbguest.auto_update = false
  end

  ###############################################################################
  # Global VirtualBox settings                                                  #
  ###############################################################################
  #config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.provider 'virtualbox' do |v|
  v.customize [
    'modifyvm', :id,
    '--groups', '/Vagrant/ansible-lab'
  ]
  end

  ###############################################################################
  # Global /etc/hosts file settings                                             #
  ###############################################################################
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  ###############################################################################
  # Global Node list                                                            #
  ###############################################################################
  require 'yaml'
  if File.file?('nodes.yaml')
    nodes = YAML.load_file('nodes.yaml')
  elsif File.file?('nodes.yaml.dist')
    nodes = YAML.load_file('nodes.yaml.dist')
  end

  ###############################################################################
  # VM definitions                                                              #
  ###############################################################################
  config.vm.define :controlnode do |controlnode|
    config.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus   = 2
    end
    controlnode.vm.host_name = "controlnode.vagrant"
    controlnode.vm.network :forwarded_port, guest: 22, host: 2230
    controlnode.vm.network :private_network, ip: "192.168.25.130"
    #ansible_host.vm.synced_folder '.', "/vagrant"
    #ansible_host.vm.provision :ansible do |ansible|
      # TODO ansible provisioning of ansible master
      # ansible.playbook = "ansible-master.yml"
    #end
  end

  nodes.each do |node|
    config.vm.define node["name"] do |srv|
      srv.vm.box                = node["box"]
      if node["box_version"]
        srv.vm.box_version      = node["box_version"]
      end
      if node["box_check_update"]
        srv.vm.box_check_update = node["box_check_update"]
      end
      srv.vm.hostname           = node["hostname"]
      if node["ports"]
        node["ports"].each do |port|
          srv.vm.network :forwarded_port, guest: port["guest"], host: port["host"]
        end
      end
      srv.vm.network :private_network, ip: node["ip"]
      if node["synced_folders"]
        node["synced_folders"].each do |folder|
          srv.vm.synced_folder folder["src"], folder["dst"]
        end
      end
      #srv.vm.provision :script do |ansible|
        # TODO ansible provisioning of nodes
        # ansible.playbook = "playbook.yml"
      #end
    end
  end
end
