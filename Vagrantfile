require 'yaml'
require 'inifile'
settings = YAML.load_file 'vagrant.yml'
default_vm_box = 'a-h/oracle_linux_7'
ansible_host_path = "ansible/hosts"

ansible_inventory = IniFile.new(:parameter => '')
ansible_inventory.filename = ansible_host_path

Vagrant.require_version ">= 1.7.0"

Vagrant.configure(2) do |config|

  nodes = settings['nodes'].length
  settings['nodes'].each_index do |index|
    vm_settings = settings['nodes'][index]
    config.vm.define "node-#{vm_settings['name']}" do |node|
      # calculate the ssh port
      host_ssh = vm_settings['ssh_host'].nil? ? (2200 + index) : vm_settings['ssh_host']
      # calculate the hostname
      host_name = vm_settings['hostname'].nil? ? "node-#{vm_settings['name']}" : vm_settings['hostname']
      # calculate the internal network ip
      ip_address = vm_settings['ip'].nil? ? "192.168.78.#{101 + index}" : vm_settings['ip']

      node.vm.box = default_vm_box if vm_settings['box'].nil?
      node.vm.hostname = host_name

      # internal network
      node.vm.network "private_network", ip: ip_address, virtualbox__intnet: true

      # set port and ssh config
      node.ssh.insert_key = false
      node.ssh.port = host_ssh
      node.ssh.insert_key = false
      # disable the default port forwarding
      node.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh", disabled: true
      # use our port forwarding
      node.vm.network "forwarded_port", guest: 22, host: host_ssh
      node.vm.synced_folder ".", "/vagrant", disabled: true

      # update the ansible inventory
      ansible_inventory['targets']["#{host_name} ansible_port=#{host_ssh}"] = nil
      # if there is an associated group this will be populated in the inventory
      if ! vm_settings['groups'].nil?
        vm_settings['groups'].each do |group|
          ansible_inventory[group]["#{host_name}"] = nil
        end
      end
      ansible_inventory.write()

      # hosts provision
      node.vm.provision "hosts" do |provisioner|
        provisioner.sync_hosts = true
        provisioner.autoconfigure = true
      end

      # ansible provision
      if index == (nodes - 1)
        node.vm.provision "ansible" do |ansible|
          ansible.verbose = "v"
          ansible.limit = "all"
          ansible.playbook = "default.yml"
          ansible.inventory_path = ansible_host_path
        end
      end
    end
  end

  #
  # Ansible Inventory common
  #
  settings['common'].each do |key, value|
    ansible_inventory['targets:vars']["#{key}=#{value}"] = nil
    ansible_inventory.write()
  end

  # Ansible group vars
  if ! settings['group_vars'].nil?
    settings['group_vars'].each do |group, vars|
      vars.each do |key, value|
        ansible_inventory["#{group}:vars"]["#{key}=#{value}"] = nil
      end
    end
    ansible_inventory.write()
  end

  #
  # Ansible proxy
  #
  if Vagrant.has_plugin?("vagrant-proxyconf")
    if ! settings['common']['proxy'].nil?
      config.proxy.http     = settings['common']['proxy']
      config.proxy.https    = settings['common']['proxy']
    else
      config.proxy.http = ''
      config.proxy.https = ''
    end
    config.proxy.no_proxy = "localhost,127.0.0.1"
  end
end
