# -*- mode: ruby -*-
# vi: set ft=ruby :
disk = '../sata1.vdi'

Vagrant.configure("2") do |config|
  config.vm.define "server-backup" do |srv|
    srv.vm.box = 'centos/7'
    srv.vm.host_name = 'server-backup'
    # Add networks
    srv.vm.network :private_network, ip: '192.168.11.160', adapter: 8
    # Change VM
    srv.vm.provider :virtualbox do |vb|
      # Change memory size
      vb.memory = "1024"
      # Create & attach disk
      unless File.exist?(disk)
        vb.customize ['createhd', '--filename', disk, '--size', 2 * 1024]
      end
        #vb.customize ['storagectl', :id, '--name', 'SATA', '--add', 'sata' ]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', disk]
      end
    srv.vm.provision "ansible" do |ansible|
            ansible.playbook = "ansible/playbooks/provision.yml"
            ansible.inventory_path = "ansible/inventory"
            ansible.host_key_checking = "false"
            ansible.limit = "all"
      end
    end
  end

Vagrant.configure("2") do |config|
  config.vm.define "client" do |cl|
    cl.vm.box = 'centos/7'
    cl.vm.host_name = 'client'
    # Add networks
    cl.vm.network 'private_network', ip: '192.168.11.150', adapter: 8
    # Change VM
    cl.vm.provider :virtualbox do |vb|
      # Change memory size
      vb.memory = "1024"
    end
  end
end
