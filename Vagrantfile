# -*- mode: ruby -*-
# vim: set ft=ruby :

# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
    :web => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "web",
        :net => [
            ["192.168.1.1", 2, "255.255.255.0","net-1"]
        ],
    },

    :elk => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "elk",
        :net => [
            ["192.168.1.2", 2, "255.255.255.0","net-1"]
        ],
    },

    :rsyslog => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "syslog",
        :net => [
            ["192.168.1.3", 2, "255.255.255.0","net-1"]
        ],
    }
}

ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'
Vagrant.configure("2") do |config|
    MACHINES.each do |boxname, boxconfig|
      config.vm.define boxname do |box|
        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxconfig[:vm_name]
        
        box.vm.provider "virtualbox" do |v|
            if boxconfig[:vm_name] == "log"
                v.memory = 4096
                v.cpus = 2
            else
                v.memory = 2048
                v.cpus = 1
            end
        end
  
        boxconfig[:net].each do |ipconf|
            box.vm.network("private_network", ip: ipconf[0], adapter: ipconf[1], netmask: ipconf[2], virtualbox__intnet: ipconf[3])
        end
  
        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
          cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL

      end
    end
  end
  
        

