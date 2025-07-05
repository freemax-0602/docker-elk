# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
    :web => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "web",
        :net => [
            ["192.168.1.1", 2, "255.255.255.0","net-1"]
        ],
        :ports => [ [8080, 8088] ],
    },

    :elk => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "elk",
        :net => [
            ["192.168.1.2", 2, "255.255.255.0","net-1"]
        ],
        :ports => [ [5601,5602] ]
    }
}

ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'
Vagrant.configure("2") do |config|
    MACHINES.each do |boxname, boxconfig|
      config.vm.define boxname do |box|
        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxconfig[:vm_name]
        
        box.vm.provider "virtualbox" do |v|
            if boxconfig[:vm_name] == "elk"
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

        if boxconfig.key?(:ports)
            boxconfig[:ports].each do |guest, host|
                box.vm.network "forwarded_port", guest: guest, host: host
            end
        end

        box.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh
            cp ~vagrant/.ssh/auth* ~root/.ssh
            sudo sed -i 's/\#PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
            systemctl restart sshd
        SHELL

      end
    end
end
  
        

