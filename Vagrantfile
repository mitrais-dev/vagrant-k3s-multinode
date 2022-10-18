# -*- mode: ruby -*-
# vi: set ft=ruby :

BOX_BASE = "ubuntu/focal64" # VM Operating System
BOX_CPU = 2 # Number of virtual cpu assigned to VM
BOX_MEMORY = "1024" # Amount of memory assigned to VM
MASTER_IP = "192.168.2.101" # Control Plane IP Address
NODE_IP = "192.168.2.102" # Node IP Address
TOKEN = "12345" # Shared secret used to join a server or agent to a cluster 

Vagrant.configure("2") do |config|
  config.vm.define "master" do |master|
    master.vm.box = BOX_BASE
    master.vm.network "private_network", ip: MASTER_IP
    master.vm.hostname = "k3s-master"
    master.vm.provider "virtualbox" do |vb|
      vb.memory = BOX_MEMORY
      vb.cpus = BOX_CPU
    end
    master.vm.provision "shell", inline: $install_server
    master.vm.provision "shell", inline: "cp /etc/rancher/k3s/k3s.yaml /vagrant/"
    master.vm.provision "shell", inline: "cat /var/lib/rancher/k3s/server/token > /vagrant/token"
  end
  config.vm.define "node1" do |node1|
    node1.vm.box = BOX_BASE
    node1.vm.network "private_network", ip: NODE_IP
    node1.vm.hostname = "k3s-node1"
    node1.vm.provider "virtualbox" do |vb|
      vb.memory = BOX_MEMORY
      vb.cpus = BOX_CPU
    end
    node1.vm.provision "shell", inline: $install_agent
  end
end

$install_server = <<-SCRIPT
curl -sfL https://get.k3s.io | \
  K3S_KUBECONFIG_MODE="644" \
  INSTALL_K3S_EXEC="--flannel-backend host-gw --node-ip #{MASTER_IP} --tls-san #{MASTER_IP} --advertise-address #{MASTER_IP} --node-external-ip #{MASTER_IP} --flannel-iface enp0s8" \
  sh -
SCRIPT

$install_agent = <<-SCRIPT
curl -sfL https://get.k3s.io | K3S_URL=https://#{MASTER_IP}:6443 \
  K3S_KUBECONFIG_MODE="644" \
  K3S_TOKEN=$(cat /vagrant/token) \
  INSTALL_K3S_EXEC="--node-ip #{NODE_IP} --node-external-ip #{NODE_IP} --flannel-iface enp0s8" sh -
SCRIPT