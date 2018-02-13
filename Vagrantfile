# -*- mode: ruby -*-
# vi: set ft=ruby :

nodes = [
  { :hostname => 'worker-01', :ip => '192.168.33.10', :ram => 512 },
  { :hostname => 'worker-02', :ip => '192.168.33.11', :ram => 512 },
  { :hostname => 'worker-03', :ip => '192.168.33.12', :ram => 512 },
  { :hostname => 'proxy-01', :ip => '192.168.33.100', :ram => 256, :forward => true}
]

Vagrant.configure("2") do |config|
  nodes.each do |node|
    config.vm.define node[:hostname] do |cfg|
      cfg.vm.box = "ubuntu/trusty64";
      cfg.vm.hostname = node[:hostname]
      cfg.vm.network :private_network, ip: node[:ip]
      # the proxy node is the only one reachable in port 8080
      if node[:forward]
        cfg.vm.network "forwarded_port", guest: 8080, host: 8080
      end
      memory = node[:ram]
      cfg.vm.provider :virtualbox do |vb|
        vb.customize [
          "modifyvm", :id,
          "--memory", memory.to_s,
          "--cpus", "1"
        ]
      end
      cfg.vm.provision "ansible_local" do |ansible|
        ansible.install = true
        ansible.playbook = "playbook.yml"
        ansible.compatibility_mode = "2.0"

        # consul/nomad/haproxy will be listening on eth1
        ansible.extra_vars = {
          network_private_address: "{{ ansible_eth1.ipv4.address }}"
        }
        ansible.groups = {
          "worker" => ["worker-01", "worker-02", "worker-03"],
          "deploy" => ["worker-03"],
          "proxy"  => ["proxy-01"]
        }
      end
    end
  end
end
