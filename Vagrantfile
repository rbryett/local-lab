VAGRANT_API_VERSION = "2"

BENTO_CENTOS_BOX = "bento/centos-7.7"
BENTO_CENTOS_VERSION = "202005.12.0"

HOST_DOMAIN = "test.local"

cluster = [
  { :hostname => "consul01", :ip => "172.17.8.101", :box => "#{BENTO_CENTOS_BOX}", :version => "#{BENTO_CENTOS_VERSION}", :cpu => "1", :ram => "2048", :forwarded_ports => [{ :guest => 8080, :host => 8081 }] },
  { :hostname => "consul02", :ip => "172.17.8.102", :box => "#{BENTO_CENTOS_BOX}", :version => "#{BENTO_CENTOS_VERSION}", :cpu => "1", :ram => "2048", :forwarded_ports => [{ :guest => 8080, :host => 8082 }] },
  { :hostname => "consul03", :ip => "172.17.8.103", :box => "#{BENTO_CENTOS_BOX}", :version => "#{BENTO_CENTOS_VERSION}", :cpu => "1", :ram => "2048", :forwarded_ports => [{ :guest => 8080, :host => 8083 }] },

  { :hostname => "vault01", :ip => "172.17.8.111", :box => "#{BENTO_CENTOS_BOX}", :version => "#{BENTO_CENTOS_VERSION}", :cpu => "1", :ram => "2048", :forwarded_ports => [{ :guest => 8200, :host => 8201 }] },
  { :hostname => "vault02", :ip => "172.17.8.112", :box => "#{BENTO_CENTOS_BOX}", :version => "#{BENTO_CENTOS_VERSION}", :cpu => "1", :ram => "2048", :forwarded_ports => [{ :guest => 8200, :host => 8202 }] },
  { :hostname => "vault03", :ip => "172.17.8.113", :box => "#{BENTO_CENTOS_BOX}", :version => "#{BENTO_CENTOS_VERSION}", :cpu => "1", :ram => "2048", :forwarded_ports => [{ :guest => 8200, :host => 8203 }] }
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

  cluster.each do |host|
    config.vm.define host[:hostname] do |host_config|
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

      host_config.vm.provision :ansible_local do |ansible|
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
          ]
        }

        ansible.playbook = "ansible/site.yml"

        ansible.verbose = "-vv"

      end
    end
  end
end
