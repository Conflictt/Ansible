# -*- mode: ruby -*-
# vim: set ft=ruby :

ENV["LC_ALL"] = "en_US.UTF-8"

Vagrant.configure("2") do |config|

  config.vm.box = "Conflict/Debian11"
  config.vm.box_check_update = false
  
  config.vm.define "03-Ansible" do |ansible|
    ansible.vm.hostname = "03-Ansible"
    ansible.vm.network "public_network", ip: "192.168.88.238"

    ansible.vm.provider "virtualbox" do |vb|
      vb.check_guest_additions = false
      vb.name = "03-Ansible" 
      vb.gui = false
      vb.memory = 1024
      vb.cpus = 1
    end
    ansible.vm.provision :ansible do |ansible|
      ansible.limit = "all"
      ansible.playbook = "nginx.yml"
    end
  end
end