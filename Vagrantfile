# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 2.0.0"

####### Install required Vagrant plugin
required_plugins    = %w(vagrant-hostmanager)
plugin_installed    = false

required_plugins.each do |plugin|
  unless Vagrant.has_plugin?(plugin)
    system "vagrant plugin install #{plugin}"
    plugin_installed = true
  end
end

# If new plugins installed, restart Vagrant process
if plugin_installed === true
  exec "vagrant #{ARGV.join' '}"
end
########

VM_NAME        = "<template>"
VM_HOSTNAME    = "<template>-local.adepdev.com"

# All Vagrant configuration is done below.
Vagrant.configure("2") do |config|

  # Select OS
  config.vm.box = "bento/ubuntu-18.04"

  # source: https://github.com/devopsgroup-io/vagrant-hostmanager
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true


  config.vm.define "<template>-local.adepdev.com", primary: true do |server|

      # global Virtualbox config
      # Docs: https://www.vagrantup.com/docs/virtualbox/configuration.html
      # Docs: https://www.virtualbox.org/manual/UserManual.html#vboxmanage-modifyvm
      server.vm.provider :virtualbox do |v|
        v.name    = VM_NAME
        v.memory  = 512
        v.cpus    = 1
        v.gui     = false
        v.customize [
          "modifyvm", :id,
          "--natdnshostresolver1", "on",      # allow resovle hosts from hosts hostfile
          "--usbehci", "off",                 # turn off usb2.0
          "--vram", 128,                      # Set GPU memory
          "--graphicscontroller", "vmsvga"    # Set contrller that is best for mac / linux
        ]
      end

    ####### vagrant hostmanager config
    server.vm.hostname = VM_HOSTNAME
    server.vm.network :private_network, type: "dhcp"
    server.hostmanager.aliases = %w(<template>-local.adepdev.com <template>-local)

   #
   # vagrant ssh -c allows you to run an script in ssh.
   # source: https://www.vagrantup.com/docs/cli/ssh.html
   # get the dhcp address and assign it to the hosts hostfile
   #
    server.hostmanager.ip_resolver = proc do |vm, resolving_vm|
      if hostname = (vm.ssh_info && vm.ssh_info[:host])
        `vagrant ssh -c "hostname -I"`.split.last
      end
    end
    #######
  end # end gui


  ####### PROVISION
  #
  # Run Ansible from Varant host
  #
  # if windows run ansible from separate location
  if Vagrant::Util::Platform.windows? then
    ### windwos
  else
    config.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = '2.0'
      ansible.inventory_path = "ansible/inventories/dev"
      ansible.playbook = "ansible/provision.yml"
    end
  end # end if

  #######
  # fix for file uplaod permissions w/ vagrant
  config.vm.synced_folder "./", "/vagrant", mount_options: ["dmode=777,fmode=666"]

end
