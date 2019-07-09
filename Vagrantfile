require "yaml"
require "fileutils"

Vagrant.require_version ">= 1.7.0"

CONFIG = File.expand_path("./vagrant_config.rb")
if File.exist?(CONFIG)
  require CONFIG
end

# Create 3 VMs
Vagrant.configure("2") do |config|
  config.vm.box = $os_image

  (1..3).each do |i|
    config.vm.define "k8s-master#{i}", autostart:true do |node|

      node.vm.hostname="k8s-master#{i}"
      ip_addr = "#{$private_subnet}.1#{i}"
      node.vm.network "private_network",  ip: "#{ip_addr}",  auto_config: true
      node.vm.provider $provider do |v|
        v.name = "k8s-master#{i}"
        v.memory = $memory
        v.cpus = $cpus
      end
    end
  end
end

# Run ansible to create inventory file and run sample playbook.yml
Vagrant.configure("3") do |ansible|
  ansible.vm.provision "ansible" do |ansible|
    ansible.playbook = "ansible/playbook.yml"
    ansible.host_vars = {
      "k8s-master1" => { "ansible_host" => "172.16.35.11", "ansible_port" => "22"},
      "k8s-master2" => { "ansible_host" => "172.16.35.12", "ansible_port" => "22"},
      "k8s-master3" => { "ansible_host" => "172.16.35.13", "ansible_port" => "22"}
    }
    ansible.groups = {
      "k8s-main" => ["k8s-master1"],
      "k8s-masters" => ["k8s-master[2:3]"],
      "all_groups:children" => ["k8s-main", "k8s-masters"]
    }
    ansible.become = true
    ansible.limit = "all"
    ansible.host_key_checking = false
  end
end

