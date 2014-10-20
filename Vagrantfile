# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

# Make sure ansible-galaxy roles are installed
`ansible-galaxy install -r ./provisioning/Rolefile --force`

###################
# Box Definitions #
###################

boxes = [
  {
    :name => "bouncex-www-local",
    :hostname => "local.www.bounceexchange.com",
    :box => "bouncex/lamp",
    :primary => true,
    :ip => "192.168.20.2",
    :guest_ssh_port => 2645,
    :host_ssh_port => 2620,
    :memory => 1024,
    :cpus => 2,
    :guest_folder => "/var/www/vhosts/bounceexchange.com",
    :playbook => "./provisioning/playbook.yml",
    :autostart => true
  }
]

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  #################
  # Hosts Manager #
  #################
  
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true
  config.hostmanager.ip_resolver = proc do |vm|
    if vm.id
      `VBoxManage guestproperty get #{vm.id} "/VirtualBox/GuestInfo/Net/1/V4/IP"`.split()[1]
    end
  end

  ##############
  # Box Config #
  ##############

  boxes.each do |opts|
    config.vm.define opts[:name],
      primary: opts[:primary],
      autostart: opts[:autostart] do |node|

      node.vm.hostname = opts[:hostname]

      # Provider
      node.vm.provider "virtualbox" do |vb|
        vb.name = opts[:name]
        vb.memory = opts[:memory]
        vb.cpus = opts[:cpus]
      end

      # Virtual Machine Setup
      node.vm.box = opts[:box]
      node.vm.network "private_network", ip: opts[:ip]
      node.vm.synced_folder ".", opts[:guest_folder], nfs: true
      node.vm.network "forwarded_port",
        guest: opts[:guest_ssh_port],
        host: opts[:host_ssh_port],
        id: "ssh"

      # Provisioning
      node.vm.provision "ansible" do |ansible|
        ansible.playbook = opts[:playbook]
      end

      # SSH
      node.ssh.port = opts[:host_ssh_port]
      node.ssh.private_key_path = "~/.ssh/bx_vagrant"
      node.ssh.forward_agent = true
      node.ssh.username = "dev"
    end
  end
end
