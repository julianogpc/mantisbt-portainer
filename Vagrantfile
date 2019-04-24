# -*- mode: ruby -*-
# vi: set ft=ruby :

$number_of_nodes = 1
$vm_mem = "4096"
$vb_gui = false
$ansible_install_mode = "pip"
$ansible_version = "2.8.0b1"
$project_src = "/vagrant/"
$compatibility_mode = "2.0"
$ip_range="172.19.7."


# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

Vagrant.require_version ">= 2.0.2"

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.box_version = "20190327.0.0"

  required_plugins = %w(vagrant-vbguest...)
  plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
  if not plugins_to_install.empty?
    config.vbguest.no_install = true
    config.vbguest.auto_update = false
  end

  (1..$number_of_nodes).each do |i|
    hostname = "ubuntu-portainer-%02d" % i

    config.vm.define hostname do |node|
        node.vm.provider "virtualbox" do |vb|
            vb.memory = $vm_mem
            vb.gui = $vb_gui
        end
      
        ip = $ip_range + "#{i+100}"

        node.vm.network "private_network", ip: ip

        node.vm.network "forwarded_port", guest: 80, host: 8080
        node.vm.network "forwarded_port", guest: 443, host: 8443
        node.vm.network "forwarded_port", guest: 3000, host: 3000
        node.vm.network "forwarded_port", guest: 3306, host: 3306
        node.vm.network "forwarded_port", guest: 5601, host: 5601
        node.vm.network "forwarded_port", guest: 8080, host: 8080
        node.vm.network "forwarded_port", guest: 9000, host: 9000

        node.vm.hostname = hostname

        node.vm.synced_folder ".", "/vagrant", type: "virtualbox", :mount_options => ["dmode=775","fmode=644"]

        node.vm.provision "install", type: "ansible_local" do |ansible|
          ansible.playbook = "install.yml"
          ansible.install_mode = $ansible_install_mode
          ansible.version = $ansible_version
          ansible.compatibility_mode = $compatibility_mode
          ansible.extra_vars = {
            ip_range: $ip_range
          }
        end

        node.vm.provision "config", type: "ansible_local" do |ansible|
          ansible.playbook = "config.yml"
          ansible.install_mode = $ansible_install_mode
          ansible.version = $ansible_version
          ansible.compatibility_mode = $compatibility_mode
          ansible.extra_vars = {
            ip_range: $ip_range
          }
        end

        node.vm.provision "monitoring", type: "ansible_local", run: "never" do |ansible|
          ansible.playbook = "monitoring.yml"
          ansible.install_mode = $ansible_install_mode
          ansible.version = $ansible_version
          ansible.compatibility_mode = $compatibility_mode
          ansible.extra_vars = {
            ip_range: $ip_range,
            monitoring: "elk"
          }
        end

          node.vm.provision "deploy", type: "ansible_local", run: "never" do |ansible|
            ansible.playbook = "deploy.yml"
            ansible.install_mode = $ansible_install_mode
            ansible.version = $ansible_version
            ansible.compatibility_mode = $compatibility_mode
            ansible.extra_vars = {
              ip_range: $ip_range
            }
        end
    end
  end
end
