# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_NO_PARALLEL'] = 'yes'
MasterCount = 3
NodeCount = 1
LoadBalancerCount = 2
IP_NW = "172.16.16."
MASTER_IP_START = 100
NODE_IP_START = 200
LB_IP_START = 50

# Sets up hosts file and DNS
def setup_dns(node)
  # Set up /etc/hosts
  node.vm.provision "setup-hosts", :type => "shell", :path => "setup-hosts.sh" do |s|
    s.args = ["eth1", node.vm.hostname]
  end
  # Set up DNS resolution
  node.vm.provision "setup-dns", type: "shell", :path => "update-dns.sh"
end

Vagrant.configure(2) do |config|
  config.vm.provision "shell", path: "bootstrap.sh"
  config.vm.provision "shell" do |s|
    ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
    s.inline = <<-SHELL
      echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
      echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
    SHELL
  end

  # Load Balancer Node
  (1..LoadBalancerCount).each do |i|
    config.vm.define "loadbalancer#{i}" do |lb|
      lb.vm.box = "bento/ubuntu-20.04"
      lb.vm.box_check_update  = false
      # lb.vm.box_version = "3.3.0"
      lb.vm.hostname = "loadbalancer#{i}.example.com"
      lb.vm.network "private_network", ip: IP_NW + "#{LB_IP_START + i}"
      # lb.vm.network "forwarded_port", guest: 22, host: "#{2730 + i}"
      lb.vm.provider "virtualbox" do |v|
        v.name = "loadbalancer#{i}"
        v.memory = 512
        v.cpus = 1
      end
      setup_dns lb
      lb.vm.provider :libvirt do |v|
        v.memory  = 512
        v.cpus    = 1
      end
    end
  end

  # Kubernetes Master Nodes
  (1..MasterCount).each do |i|
    config.vm.define "kmaster#{i}" do |masternode|
      masternode.vm.box = "bento/ubuntu-20.04"
      masternode.vm.box_check_update  = false
      # masternode.vm.box_version = "3.3.0"
      masternode.vm.hostname = "kmaster#{i}.example.com"
      masternode.vm.network "private_network", ip: IP_NW + "#{MASTER_IP_START + i}"
      # masternode.vm.network "forwarded_port", guest: 22, host: "#{2710 + i}"
      masternode.vm.provider "virtualbox" do |v|
        v.name = "kmaster#{i}"
        v.memory = 2048
        v.cpus = 2
      end
      setup_dns masternode
    end
  end


  # Kubernetes Worker Nodes
  (1..NodeCount).each do |i|
    config.vm.define "kworker#{i}" do |workernode|
      workernode.vm.box = "bento/ubuntu-20.04"
      workernode.vm.box_check_update  = false
      # workernode.vm.box_version = "3.3.0"
      workernode.vm.hostname = "kworker#{i}.example.com"
      workernode.vm.network "private_network", ip: IP_NW + "#{NODE_IP_START + i}"
      # workernode.vm.network "forwarded_port", guest: 22, host: "#{2720 + i}"
      workernode.vm.provider "virtualbox" do |v|
        v.name = "kworker#{i}"
        v.memory = 1024
        v.cpus = 1
      end
      setup_dns workernode
    end
  end

end
