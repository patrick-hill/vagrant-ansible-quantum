# -*- mode: ruby -*-
# vi: set ft=ruby :

#####################################################
#              REQUIREMENTS                         #
#####################################################
#

#####################################################
#              VARIABLES                            #
#####################################################
# Configs
Vagrant.require_version ">= 1.7.0"
VAGRANTFILE_API_VERSION = "2"
DOMAIN          = ".hpcw.com"
NETWORK         = "11.0.0."
NETMASK         = "255.255.255.0"
BOX_CENT_6      = "hpcw-centos67-nocm-0.0.1"
BOX_CENT_7      = "hpcw-centos71-nocm-0.0.3"
BOX_UBUNTU_14   = "hpcw-ubuntu1404-desktop-nocm-0.1.0"
BOX_UBUNTU_15   = "hpcw-ubuntu1510-nocm-0.1.0"
BOX_MINT_18     = "hpcw-linuxmint18.1-desktop-nocm-0.1.0"
SERVERS = [
  {
    :hostname     => 'quantum',
    :ip           => NETWORK + '2',
    :box          => BOX_UBUNTU_15,
    :box_updates  => true,
    :ram          => 2048,
    :cpu          => 2,
    :gui          => true,
    :playbook     => 'playbook.yml',  # Gets appended to playbook-<var_name>.yml
    :ports        => []
  }
]


Vagrant.configure("2") do |config|
  SERVERS.each do |machine|
    config.vm.define machine[:hostname] do |node|
      # The most common configuration options are documented and commented below.
      # For a complete reference, please see the online documentation at
      # https://docs.vagrantup.com.
      #####################################################
      #              PROVIDERS                            #
      #####################################################
      node.vm.provider "virtualbox" do |v|
        v.linked_clone  = true
        v.gui           = machine[:gui]
        v.memory        = machine[:ram]
        v.cpus          = machine[:cpu]
        # v.customize ["modifyvm", :id, "--rtcuseutc", "on"]
      end
      node.vm.provider "wmware_workstation" do |v|
        v.gui     = machine[:gui]
        v.memory  = machine[:ram]
        v.cpus    = machine[:cpu]
        v.vmx["ethernet0.virtualDev"] = "vmxnet3"
      end
      #####################################################
      #              BOX CONFIG                           #
      #####################################################
      node.vm.box               = machine[:box]
      node.vm.box_check_update  = machine[:box_updates]
      node.vm.hostname          = machine[:hostname] + DOMAIN
      # Ports for accessing the box services
      machine[:ports].each do |port|
        node.vm.network :forwarded_port, guest: port[:guest], host: port[:host], auto_correct: true
      end
      # Private net is generally desired
      node.vm.network :private_network, ip: machine[:ip],
        netmask: NETMASK,
        virtualbox__intnet: true
      # Public Net if you need other to access the box
      # node.vm.network "public_network", auto_config: false
      # Need to share another dir other than project root(default)?
      # node.vm.synced_folder "../data", "/vagrant_data"
      #####################################################
      #              PROVISIONING                         #
      #####################################################
      # https://www.vagrantup.com/docs/provisioning/ansible.html

      # Shell: Inline
    #   node.vm.provision :shell, inline: ""
    #   node.vm.provision "shell", path: <path to script>
    #   node.vm.provision :shell, inline: <<-SHELL
    #     ifconfig
    #   SHELL

      # Ansible
      node.vm.provision "ansible" do |ansible|
        # Options
        ansible.force_remote_user = 'vagrant'
        # ansible.limit = 'all'           # Disable default limit to connect to all machines
        ansible.ask_sudo_pass = false   # Prompt for sudo pass prior to play run
        ansible.ask_vault_pass = false   # Prompt for vault encryption password
        ansible.vault_password_file = 'ansible/vault-password.txt'
        # Ansible Galaxy
        ansible.galaxy_roles_path = 'ansible/roles/:../'
        ansible.galaxy_role_file = "ansible/roles/requirements.yml"
        ansible.galaxy_command = 'ansible-galaxy install --role-file=%{role_file} --roles-path=%{roles_path} --ignore-errors'
        # Ansible Playbook
        # ansible.verbose   = "v" # Vagrant displays ansible commands
        ansible.playbook  = "ansible/#{machine[:playbook]}"
        # Testing / speeding up / etc
        ansible.extra_vars = {}
      end

    end # Nodes
  end # Machines
  # ToDos:
  # - Test & Confirm registration for Satellite

  # Resources:
  # MultiMachine Vagrant File: http://stackoverflow.com/questions/24072916/multi-vm-in-one-vagrantfile-could-i-set-different-memory-size-for-each-of-them
  # Vagrant's Ansible Docs: http://docs.vagrantup.com/v2/provisioning/ansible.html
  # Vagrant Automatic Inventory File: .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory
  # Vagrant 1.7 auto Private Key Location: .vagrant/machines/[machine name]/[provider]/private_key
  # Ruby Maps: https://www.thoughtco.com/how-to-create-hashes-2908196
end
