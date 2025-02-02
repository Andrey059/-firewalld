# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
:inetRouter => {
        :box_name => "centos/6",
        #:public => {:ip => '10.10.10.1', :adapter => 1},
        :net => [
                   {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                ]
  },
:inetRouter2 => {
        :box_name => "centos/6",
        :net => [
                   {ip: '192.168.253.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                   {ip: '192.168.11.171', adapter: 3, netmask: "255.255.255.0"},
                ]
  },
:centralRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},
                   {ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "mgt-net"},
                ]
  },
:centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {adapter: 3, auto_config: false, virtualbox__intnet: true},
                   {adapter: 4, auto_config: false, virtualbox__intnet: true},
                ]
  },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "256"]
        end

        box.vm.box = boxconfig[:box_name]

        box.vm.host_name = boxname.to_s

        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end

        if boxconfig.key?(:public)
          box.vm.network "public_network", boxconfig[:public]
        end

        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
          cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL

        case boxname.to_s
        when "inetRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            #iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
	    #iptables-save > /etc/firewall.conf
    	    #echo "/sbin/iptables-restore < /etc/firewall.conf" >> /etc/rc.d/rc.local && chmod +x /etc/rc.d/rc.local
            iptables-restore < /vagrant/iptables.rules
            sysctl net.ipv4.conf.all.forwarding=1
            ip route add 192.168.0.0/24 via 192.168.255.2
            ip route add 192.168.1.0/24 via 192.168.255.2
            ip route add 192.168.2.0/24 via 192.168.255.2
            ip route add 192.168.253.0/24 via 192.168.255.2
            sed -i '66s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
            service sshd restart
            SHELL
        when "inetRouter2"
          #config.vm.network "forwarded_port", guest: 8080, host: 8080
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
	    iptables-save > /etc/firewall.conf
    	    echo "/sbin/iptables-restore < /etc/firewall.conf" >> /etc/rc.d/rc.local && chmod +x /etc/rc.d/rc.local
            sysctl net.ipv4.conf.all.forwarding=1
            ip route add 192.168.0.0/24 via 192.168.253.2
            ip route add 192.168.1.0/24 via 192.168.253.2
            ip route add 192.168.2.0/24 via 192.168.253.2
            ip route add 192.168.255.0/24 via 192.168.253.2
            iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.0.2:80
            iptables -t nat -A POSTROUTING -p tcp -d 192.168.0.2 --dport 80 -j SNAT --to-source 192.168.253.1
            SHELL
        when "centralRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            yum install -y mc traceroute nmap
            sysctl net.ipv4.conf.all.forwarding=1
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            echo -e "BOOTPROTO=static\nONBOOT=yes\nIPADDR=192.168.251.1\nNETMASK=255.255.255.252\nDEVICE=eth1:1" > /etc/sysconfig/network-scripts/ifcfg-eth1:1
            echo -e "BOOTPROTO=static\nONBOOT=yes\nIPADDR=192.168.252.1\nNETMASK=255.255.255.252\nDEVICE=eth1:2" > /etc/sysconfig/network-scripts/ifcfg-eth1:2
            echo -e "BOOTPROTO=static\nONBOOT=yes\nIPADDR=192.168.253.2\nNETMASK=255.255.255.252\nDEVICE=eth1:3" > /etc/sysconfig/network-scripts/ifcfg-eth1:3
            ifup eth1:1
            ifup eth1:2
            ifup eth1:3
            systemctl restart network
            ip route del default
            ip route add default via 192.168.255.1 dev eth1 metric 99
            ip route add 192.168.1.0/24 via 192.168.251.2
            ip route add 192.168.2.0/24 via 192.168.252.2
            ip route add 192.168.253.0/24 via 192.168.253.1
            SHELL
        when "centralServer"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            yum install -y epel-release
            yum install -y nginx mc traceroute nmap
            sed -i 's/listen\ \ \ \ \ \ \ 80\ default\_server\;/listen\ \ \ \ \ \ \ 0\.0\.0\.0\:80\ default\_server\;/g' /etc/nginx/nginx.conf
            systemctl start nginx
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            ip route del default
            ip route add default via 192.168.0.1 dev eth1 metric 99
            ip route add 192.168.0.0/24 via 192.168.0.1
            ip route add 192.168.1.0/24 via 192.168.0.1
            ip route add 192.168.2.0/24 via 192.168.0.1
            ip route add 192.168.253.0/24 via 192.168.0.1
            SHELL
        end
      end

  end

end
