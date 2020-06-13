# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.hostname = "rpm"

  config.vm.box_check_update = false

  config.vm.provision "shell", path: "rpmm.sh"
end
