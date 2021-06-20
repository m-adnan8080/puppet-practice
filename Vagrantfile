# -*- mode: ruby -*-
# vi: set ft=ruby :

cluster = {
 "puppetmaster" => { :ip => "192.168.18.111", :mem => 4096, :cpu => 2 },
#  "puppetagent1" => { :ip => "192.168.18.112", :mem => 1024, :cpu => 2 },
#  "puppetagent2" => { :ip => "192.168.18.113", :mem => 1024, :cpu => 2 }
}

Vagrant.configure("2") do |config|

  config.vm.provision "shell", inline: "echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKBYP8WLbAudznHNukAktWyDL1I0N5Wn8hus/re3ncI8' >> /home/vagrant/.ssh/authorized_keys"
  config.vm.provision "shell",
    run: "always",
    inline: "ip route add default via 192.168.18.1"

  cluster.each_with_index do |(hostname, info), index|

    if "#{hostname}" == "puppetagent2" then
      config.vm.define hostname do |cfg|
        cfg.vm.box = "centos/7"
        cfg.vm.provider :virtualbox do |vb, override|
          override.vm.network :public_network, ip: "#{info[:ip]}", bridge: "wlp0s20f3"
          override.vm.hostname = hostname + ".test"
          vb.name = hostname
          vb.customize ["modifyvm", :id, "--memory", info[:mem], "--cpus", info[:cpu], "--hwvirtex", "on"]
        end # end provider
      end # end config
    else
      config.vm.define hostname do |cfg|
        cfg.vm.box = "ubuntu/focal64"
        cfg.vm.provider :virtualbox do |vb, override|
          override.vm.network :public_network, ip: "#{info[:ip]}", bridge: "wlp0s20f3"
          override.vm.hostname = hostname + ".test"
          vb.name = hostname
          vb.customize ["modifyvm", :id, "--memory", info[:mem], "--cpus", info[:cpu], "--hwvirtex", "on"]
        end # end provider
      end # end config
    end
  end # end cluster

end
