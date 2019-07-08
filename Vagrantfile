require 'yaml'

Vagrant.require_version ">= 2.2.0"

current_dir    = File.dirname(File.expand_path(__FILE__))
configs        = YAML.load_file("#{current_dir}/vagrant_config.yaml")
vagrant_config = configs['configs'][configs['configs']['use']]

if File.exists?(configs)
  require configs
end

$os_image = (ENV['OS_IMAGE'] || "centos7").to_sym
$provider = (ENV['PROVIDER'] || "virtualbox").to_sym

def set_vbox(vb, config)
  vb.gui = false
  vb.memory = vagrant_config['system_memory']
  vb.cpus = vagrant_config['system_vcpus']

  case $os_image
  when :centos7
    config.vm.box = "bento/centos-7"
  when :ubuntu16
    config.vm.box = "bento/ubuntu-16.04"
  end
end

def set_libvirt(lv, config)
  lv.nested = true
  lv.volume_cache = 'none'
  lv.uri = 'qemu+unix:///system'
  lv.memory = vagrant_config['system_memory']
  lv.cpus = vagrant_config['system_vcpus']

  case $os_image
  when :centos7
    config.vm.box = "centos/7"
  when :ubuntu16
    config.vm.box = "yk0/ubuntu-xenial"
  end
end

def set_hyperv(hv, config)
  hv.memory = vagrant_config['system_memory']
  hv.cpus = vagrant_config['system_vcpus']

  case $os_image
  when :centos7
    config.vm.box = "generic/centos7"
    config.vm.provision "shell", inline: "sudo yum install -y python"
  when :ubuntu16
    config.vm.box = "generic/ubuntu1604"
    config.vm.provision "shell", inline: "sudo apt-get update && sudo apt-get install -y python"
  end
end

Vagrant.configure("2") do |config|
  config.vm.provider "hyperv"
  config.vm.provider "virtualbox"
  config.vm.provider "libvirt"

  config.vm.provision "shell", inline: "sudo swapoff -a"
  if $provider.to_s != 'hyperv'
    config.vm.provision "shell", inline: "sudo cp /vagrant/hosts /etc/"
  end

  #count = $net_count
  count = 1
  (1..(vagrant_config['master_count'] + vagrant_config['node_count]')).each do |mid|
    name = (mid <= vagrant_config['master_count']) ? "k8s-m" : "k8s-n"
    id   = (mid <= vagrant_config['master_count']) ? mid : (mid - vagrant_config['master_count'])

    config.vm.define "#{name}#{id}" do |n|
      n.vm.hostname = "#{name}#{id}"
      ip_addr = "#{vagrant_config['private_subnet]'}.#{count}"
      n.vm.network :private_network, ip: "#{ip_addr}",  auto_config: true
      if vagrant_config['bridge_enable'] && vagrant_config['bridge_eth'].to_s != ''
        n.vm.network "public_network", bridge: vagrant_config['bridge_eth']
      end

      # Configure virtualbox provider
      n.vm.provider :virtualbox do |vb, override|
        vb.name = "#{n.vm.hostname}"
        set_vbox(vb, override)
      end

      # Configure libvirt provider
      n.vm.provider :libvirt do |lv, override|
        lv.host = "#{n.vm.hostname}"
        set_libvirt(lv, override)
      end

      # Configure hyperv provider
      n.vm.provider :hyperv do |hv, override|
        hv.vmname = "#{n.vm.hostname}"
        set_hyperv(hv, override)
      end

      count += 1
      if mid == (vagrant_config['master_count'] + vagrant_config['node_count']) && vagrant_config['provider'].to_s != 'hyperv' && vagrant_config['auto_deploy'].to_s == 'true'
        n.vm.provision "cluster", type: "ansible" do |ansible|
          ansible.playbook = "cluster.yml"
          ansible.inventory_path = "./inventory/hosts.ini"
          ansible.become = true
          ansible.limit = "all"
          ansible.host_key_checking = false
        end
        n.vm.provision "addon", type: "ansible" do |ansible|
          ansible.playbook = "addons.yml"
          ansible.inventory_path = "./inventory/hosts.ini"
          ansible.become = true
          ansible.limit = "all"
          ansible.host_key_checking = false
        end
      end
    end
  end
end
