# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANT_BOX = "ubuntu/bionic64"
VM_CPUS = 2
VM_MEMORY = 2048
INSTANCES = 3

# Controller config.
$controller_hostname = "k8s-1"
$controller_ip_address = "192.168.4.81"

# Sets guest environment variables.
# @see https://github.com/hashicorp/vagrant/issues/7015
$set_environment_variables = <<SCRIPT
tee "/etc/profile.d/myvars.sh" > "/dev/null" <<EOF
# Ansible environment variables.
export ANSIBLE_RETRY_FILES_ENABLED=0
EOF
SCRIPT

VAGRANT_ROOT = File.dirname(File.expand_path(__FILE__))

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  config.ssh.private_key_path = "./ansible/insecure_private_key"
  config.vm.box = VAGRANT_BOX

  (2..INSTANCES).each do |i|
    config.vm.define "k8s-#{i}" do |machine|
      machine.vm.provider "virtualbox" do |vbox|
        vbox.name = "k8s-#{i}"
        vbox.memory = VM_MEMORY
        vbox.cpus = VM_CPUS
        # Uncomment if you want to disable VT-x to use with KVM. This
        # slows down the virtual machines.
        # vbox.customize ["modifyvm", :id, "--hwvirtex", "off"]
      end
      machine.vm.hostname = "k8s-#{i}"
      machine.vm.network "private_network", ip: "192.168.4.#{80 + i}"
    end
  end

  # The controller will provision the other nodes. In Windows
  # environment there may be issues in provisioning so we add an extra
  # virtual machine to provision the others.
  #
  # It will also be our master and it will be allowed to run pods.
  config.vm.define $controller_hostname do |machine|
    machine.vm.provider "virtualbox" do |vbox|
      vbox.name = $controller_hostname
      vbox.memory = VM_MEMORY
      vbox.cpus = VM_CPUS
      # Uncomment if you want to disable VT-x to use with KVM.
      # vbox.customize ["modifyvm", :id, "--hwvirtex", "off"]
    end
    machine.vm.hostname = $controller_hostname
    machine.vm.network "private_network", ip: $controller_ip_address
    # Vault passwords in home dir in order to not leave the key together with
    # the lock (useful to synchronize projects inside Dropbox/Gdrive).
    machine.vm.synced_folder "~/.ansible_secret", \
      "/home/vagrant/.ansible_secret"
    machine.vm.synced_folder "ansible", "/etc/ansible"
    machine.vm.provision "shell", inline: $set_environment_variables, \
      run: "always"
    machine.vm.provision "shell", path: "scripts/bootstrap.sh"
    machine.vm.provision "ansible_local" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.install = false
      ansible.provisioning_path = "/etc/ansible"
      ansible.playbook = ENV["ANSIBLE_PLAYBOOK"] ? ENV["ANSIBLE_PLAYBOOK"] \
        : "playbook.yml"
      ansible.inventory_path = "hosts-dev.ini"
      ansible.become = true
      ansible.limit = ENV['ANSIBLE_LIMIT'] ? ENV['ANSIBLE_LIMIT'] : "all"
      ansible.vault_password_file = "/home/vagrant/.ansible_secret/vault_pass_insecure"
      ansible.tags = ENV['ANSIBLE_TAGS']
      ansible.verbose = ENV['ANSIBLE_VERBOSE']
    end
  end
end
