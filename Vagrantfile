# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"

  #他のデバイスからもアクセスできるようにpublic_network設定。ipも指定できるけど一旦dhcpで動的割り当て
  config.vm.network "public_network"

  #ipの設定
  #config.vm.network "private_network", ip: "192.168.33.100"

  #共有フォルダの設定
  config.vm.synced_folder "./", "/vagrant", owner: "vagrant", group: "vagrant"

  #shellの実行
  config.vm.provision :shell, :inline => "apt-get update"
  config.vm.provision :shell, :path => "provision/install_docker.sh"
  config.vm.provision :shell, :path => "provision/install_docker-compose.sh"

  config.vm.provider "virtualbox" do |vm|
    vm.memory = 4096
  end

end
