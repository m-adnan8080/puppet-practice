# -*- mode: ruby -*-
# vi: set ft=ruby :

cluster = {
  "foreman" => { :ip => "192.168.100.2", :gw => "192.168.100.1", :mem => 8192, :cpu => 2 },
  # "server1" => { :ip => "192.168.100.3", :mem => 512, :cpu => 1 },
}

Vagrant.configure("2") do |config|

  config.vm.provision "shell", inline: "echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKBYP8WLbAudznHNukAktWyDL1I0N5Wn8hus/re3ncI8' >> /home/vagrant/.ssh/authorized_keys"
  config.vm.synced_folder ".", "/vagrant", disabled: true


  cluster.each_with_index do |(hostname, info), index|

    if "#{hostname}" == "foreman" then
      config.vm.provision "shell", inline: "nmcli conn mod 'System eth1' ipv4.gateway #{info[:gw]}"
      config.vm.provision "shell", inline: "nmcli conn up 'System eth1'"
      config.vm.define hostname do |cfg|
        cfg.vm.box = "centos/7"
        cfg.vm.provider :virtualbox do |vb, override|
          # override.vm.network :public_network, ip: "#{info[:ip]}", bridge: "wlp0s20f3"
          override.vm.network :private_network, ip: "#{info[:ip]}"
          override.vm.hostname = hostname + ".local.test"
          vb.name = hostname
          vb.customize ["modifyvm", :id, "--memory", info[:mem], "--cpus", info[:cpu], "--hwvirtex", "on"]
        end # end provider
      end # end config
    else
      config.vm.define hostname do |cfg|
        cfg.vm.box = "ubuntu/focal64"
        cfg.vm.provider :virtualbox do |vb, override|
          # override.vm.network :public_network, ip: "#{info[:ip]}", bridge: "wlp0s20f3"
          override.vm.network :private_network, type: dhcp
          override.vm.hostname = hostname + ".local.test"
          vb.name = hostname
          vb.customize ["modifyvm", :id, "--memory", info[:mem], "--cpus", info[:cpu], "--hwvirtex", "on"]
        end # end provider
      end # end config
    end
  end # end cluster

end
