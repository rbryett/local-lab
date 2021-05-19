VAGRANT_API_VERSION = "2"

UBUNTU_BOX = "bento/ubuntu-20.04"
UBUNTU_VERSION = "202010.24.0"

HOST_DOMAIN = "test.local"

INVENTORY_PATH = "ansible/inventories/vagrant/hosts"


cluster = [
  { :hostname => "consul01", :ip => "172.17.8.101", :box => "#{UBUNTU_BOX}", :version => "#{UBUNTU_VERSION}", :cpu => "1", :ram => "2048", :forwarded_ports => [{ :guest => 8500, :host => 8081 }] },
  { :hostname => "consul02", :ip => "172.17.8.102", :box => "#{UBUNTU_BOX}", :version => "#{UBUNTU_VERSION}", :cpu => "1", :ram => "2048", :forwarded_ports => [{ :guest => 8500, :host => 8082 }] },
  { :hostname => "consul03", :ip => "172.17.8.103", :box => "#{UBUNTU_BOX}", :version => "#{UBUNTU_VERSION}", :cpu => "1", :ram => "2048", :forwarded_ports => [{ :guest => 8500, :host => 8083 }] },

  { :hostname => "agent01", :ip => "172.17.8.10", :box => "#{UBUNTU_BOX}", :version => "#{UBUNTU_VERSION}", :cpu => "1", :ram => "2048", :forwarded_ports => [] },
  { :hostname => "agent02", :ip => "172.17.8.11", :box => "#{UBUNTU_BOX}", :version => "#{UBUNTU_VERSION}", :cpu => "1", :ram => "2048", :forwarded_ports => [] },

  # { :hostname => "vault01", :ip => "172.17.8.111", :box => "#{UBUNTU_BOX}", :version => "#{UBUNTU_VERSION}", :cpu => "1", :ram => "2048", :forwarded_ports => [{ :guest => 8200, :host => 8201 }] },
  # { :hostname => "vault02", :ip => "172.17.8.112", :box => "#{UBUNTU_BOX}", :version => "#{UBUNTU_VERSION}", :cpu => "1", :ram => "2048", :forwarded_ports => [{ :guest => 8200, :host => 8202 }] },
  # { :hostname => "vault03", :ip => "172.17.8.113", :box => "#{UBUNTU_BOX}", :version => "#{UBUNTU_VERSION}", :cpu => "1", :ram => "2048", :forwarded_ports => [{ :guest => 8200, :host => 8203 }] }
]

ETC_HOSTS_ENTRIES = ""
cluster.each do |host|
  ETC_HOSTS_ENTRIES << "#{host[:ip]} #{host[:hostname]}.#{HOST_DOMAIN} #{host[:hostname]}\n"
end

$ETC_HOSTS_SCRIPT = <<SCRIPT
#!/bin/bash
cat /etc/hosts << EOF
127.0.0.1 localhost localhost.localdomain
#{ETC_HOSTS_ENTRIES}
EOF

hostname --fqdn > /etc/hosname && hostname -F /etc/hostname
sed "s/^[ \t]*//" -i /etc/hosts
cat /etc/hosts

SCRIPT


Vagrant.configure(VAGRANT_API_VERSION) do |config|
  config.vm.box_check_update = false
  
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = false
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  hosts_length = cluster.length()

  cluster.each_with_index do |host, host_id|

    config.vm.define host[:hostname] do |host_config|
      # Incrememnt index for ansible if logic further down
      host_id += 1

      host_config.vm.box = host[:box]
      host_config.vm.box_version = host[:version]
      host_config.vm.network "private_network", ip: host[:ip]
      host_config.vm.hostname = "#{host[:hostname]}.#{HOST_DOMAIN}"
      host_config.hostmanager.aliases = "#{host[:hostname]}"

      host[:forwarded_ports].each do |rule|
        host_config.vm.network "forwarded_port", guest: rule[:guest], host: rule[:host]
      end

      host_config.vm.provider :virtualbox do |vbox|
        vbox.name = host[:hostname].to_s
        vbox.customize ["modifyvm", :id, "--memory", host[:ram].to_s ]
        vbox.customize ["modifyvm", :id, "--cpus", host[:cpu].to_s ]
      end

      host_config.vm.provision :shell, :inline => $ETC_HOSTS_SCRIPT

      if host_id == hosts_length
        host_config.vm.provision :ansible do |ansible|

          ansible.groups = {
            "vault" => [
              "vault01",
              "vault02",
              "vault03",
            ],
            "consul" => [
              "consul01",
              "consul02",
              "consul03"
            ],
            "consul-agents" => [
              "agent01",
              "agent02",
            ]
          }
      
          ansible.playbook = "ansible/site.yml"
          ansible.limit = "all"
          ansible.verbose = "-vv"
        end
      end
    end
  end
end
