ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

cluster = {
  "foreman" => { :ip => "192.168.100.111", :mem => 4096, :cpu => 2 },
#  "server1" => { :ip => "192.168.100.112", :mem => 512, :cpu => 1 },
#  "server2" => { :ip => "192.168.100.113", :mem => 512, :cpu => 1 },
#  "server3" => { :ip => "192.168.100.113", :mem => 512, :cpu => 1 },
#  "server4" => { :ip => "192.168.100.113", :mem => 512, :cpu => 1 },
#  "server5" => { :ip => "192.168.100.113", :mem => 512, :cpu => 1 },
#  "server6" => { :ip => "192.168.100.113", :mem => 512, :cpu => 1 },
}

Vagrant.configure("2") do |config|

  config.vm.provision "shell", inline: "echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKBYP8WLbAudznHNukAktWyDL1I0N5Wn8hus/re3ncI8' >> /home/vagrant/.ssh/authorized_keys"
  config.vm.synced_folder ".", "/vagrant", disabled: true

  cluster.each_with_index do |(node, info), index|

    if "#{node}" == "foreman" then
      config.vm.define node do |cfg|
        cfg.vm.box = "ubuntu/20.04"
        cfg.vm.hostname = node + '.local.test'
        cfg.vm.provider :libvirt do |domain|
          domain.memory = info[:mem]
          domain.cpus = info[:cpu]
        end # end provider
      end # end config
    else
      config.vm.define node do |cfg|
        cfg.vm.box = "ubuntu/20.04"
        cfg.vm.hostname = node + '.local.test'
        cfg.vm.provider :libvirt do |domain|
          domain.memory = info[:mem]
          domain.cpus = info[:cpu]
        end # end provider
      end # end config
    end
  end # end cluster

end
