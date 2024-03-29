require "yaml"
require "fileutils"

Vagrant.require_version ">= 1.7.0"

CONFIG = File.expand_path("./vagrant_config.rb")
if File.exist?(CONFIG)
  require CONFIG
end

NODES = 3

# Create 3 VMs
Vagrant.configure("2") do |node|
  node.vm.box = $os_image

  (1..NODES).each do |node_id|
    node.vm.define "k8s-master#{node_id}", autostart:true do |node|

      node.vm.hostname="k8s-master#{node_id}"
      node.vm.provision "shell", inline: "swapoff -a"
      ip_addr = "#{$private_subnet}.1#{node_id}"
      node.vm.network "private_network",  ip: "#{ip_addr}",  auto_config: true
      node.vm.provider $provider do |v|
        v.name = "k8s-master#{node_id}"
        v.memory = $memory
        v.cpus = $cpus
      end
      # only start ansible provision after the last box (does not work with ansible local Windows)
      if node_id == NODES
        node.vm.provision "ansible" do |ansible|
          ansible.playbook = "#{$ansible_playbook}"
          ansible.host_vars = {
            "k8s-master1" => { "ansible_host" => "#{$private_subnet}.11", "ansible_port" => "22"},
            "k8s-master2" => { "ansible_host" => "#{$private_subnet}.12", "ansible_port" => "22"},
            "k8s-master3" => { "ansible_host" => "#{$private_subnet}.13", "ansible_port" => "22"}
          }
          ansible.groups = {
            "master" => ["k8s-master1"],
            "masters" => ["k8s-master[1:3]"]
          }
          ansible.become = true
          ansible.limit = "all,localhost"
        end
      end
    end
  end
end
