# -*- mode: ruby -*-
# vi: set ft=ruby :

cluster = {
  "foreman" => { :ip => "192.168.100.111", :mem => 4096, :cpu => 2 },
  "server1" => { :ip => "192.168.100.112", :mem => 512, :cpu => 1 },
  "server2" => { :ip => "192.168.100.113", :mem => 512, :cpu => 1 },
  "server3" => { :ip => "192.168.100.113", :mem => 512, :cpu => 1 },
  "server4" => { :ip => "192.168.100.113", :mem => 512, :cpu => 1 },
  "server5" => { :ip => "192.168.100.113", :mem => 512, :cpu => 1 },
  "server6" => { :ip => "192.168.100.113", :mem => 512, :cpu => 1 },
}

Vagrant.configure("2") do |config|

  config.vm.provision "shell", inline: "echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKBYP8WLbAudznHNukAktWyDL1I0N5Wn8hus/re3ncI8' >> /home/vagrant/.ssh/authorized_keys"
  config.vm.synced_folder ".", "/vagrant", disabled: true

  cluster.each_with_index do |(hostname, info), index|

    if "#{hostname}" == "foreman" then
      config.vm.define hostname do |cfg|
        cfg.vm.box = "centos/7"
        cfg.vm.provider :virtualbox do |vb, override|
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
          override.vm.network :private_network, ip: "#{info[:ip]}"
          override.vm.hostname = hostname + ".local.test"
          vb.name = hostname
          vb.customize ["modifyvm", :id, "--memory", info[:mem], "--cpus", info[:cpu], "--hwvirtex", "on"]
        end # end provider
      end # end config
    end
  end # end cluster

end
