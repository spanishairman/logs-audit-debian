# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.define "Debian12-LogServer" do |logserver|
  logserver.vm.box = "/home/max/vagrant/images/debian12"
  logserver.vm.network :private_network,
       :type => 'ip',
       :libvirt__forward_mode => 'veryisolated',
       :libvirt__dhcp_enabled => false,
       :ip => '192.168.1.1',
       :libvirt__netmask => '255.255.255.248',
       :libvirt__network_name => 'vagrant-libvirt-inet1',
       :libvirt__always_destroy => false
  logserver.vm.provider "libvirt" do |lvirt|
      lvirt.memory = "1024"
      lvirt.cpus = "1"
      lvirt.title = "Debian12-inetRouter"
      lvirt.description = "Виртуальная машина на базе дистрибутива Debian Linux. LogServer"
      lvirt.management_network_name = "vagrant-libvirt-mgmt"
      lvirt.management_network_address = "192.168.121.0/24"
      lvirt.management_network_keep = "true"
      lvirt.management_network_mac = "52:54:00:27:28:83"
  end
  logserver.vm.provision "shell", inline: <<-SHELL
      brd='*************************************************************'
      echo "$brd"
      echo 'Set Hostname'
      hostnamectl set-hostname logserver
      echo "$brd"
      sed -i 's/debian12/logserver/' /etc/hosts
      sed -i 's/debian12/logserver/' /etc/hosts
      echo "$brd"
      echo 'Изменим ttl для работы через раздающий телефон'
      echo "$brd"
      sysctl -w net.ipv4.ip_default_ttl=66
      echo "$brd"
      echo 'Если ранее не были установлены, то установим необходимые  пакеты'
      echo "$brd"
      apt update
      export DEBIAN_FRONTEND=noninteractive
      apt install -y iptables iptables-persistent rsyslog auditd
      sed -i 's/#module(load="imudp")/module(load="imudp")/' /etc/rsyslog.conf
      sed -i 's/#input(type="imudp" port="514")/input(type="imudp" port="514")/' /etc/rsyslog.conf
      sed -i 's/#module(load="imtcp")/module(load="imtcp")/' /etc/rsyslog.conf
      sed -i 's/#input(type="imtcp" port="514")/input(type="imtcp" port="514")/' /etc/rsyslog.conf
      echo "# Remote Logs" >> /etc/rsyslog.conf
      echo '$template RemoteLogs,"/var/log/%HOSTNAME%/%PROGRAMNAME%.log"' >> /etc/rsyslog.conf
      echo '*.* ?RemoteLogs' >> /etc/rsyslog.conf
      echo '& stop' >> /etc/rsyslog.conf
      sed -i 's/##tcp_listen_port = 60/tcp_listen_port = 60/' /etc/audit/auditd.conf
      systemctl restart rsyslog.service
      systemctl restart auditd.service
      # Политика по умолчанию для цепочки INPUT - DROP
      iptables -P INPUT DROP
      # Политика по умолчанию для цепочки OUTPUT - DROP
      iptables -P OUTPUT DROP
      # Политика по умолчанию для цепочки FORWARD - DROP
      iptables -P FORWARD DROP
      # Базовый набор правил: разрешаем локалхост, запрещаем доступ к адресу сети обратной петли не от локалхоста, разрешаем входящие пакеты со статусом установленного сонединения.
      iptables -A INPUT -i lo -j ACCEPT
      iptables -A INPUT ! -i lo -d 127.0.0.0/8 -j REJECT
      iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
      # Разрешим транзит трафика.
      # iptables -A FORWARD -j ACCEPT
      # Открываем исходящие
      iptables -A OUTPUT -j ACCEPT
      # Разрешим входящие с хоста управления.
      iptables -A INPUT -s 192.168.121.1 -j ACCEPT
      # Также, разрешим входящие для WebServer и AppServer
      iptables -A INPUT -s 192.168.1.0/29 -m tcp -m multiport -p tcp --dports 60,514 -j ACCEPT
      iptables -A INPUT -s 192.168.1.0/29 -m udp -p udp --dport 514 -j ACCEPT
      # Откроем ICMP ping
      iptables -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
      netfilter-persistent save
      # echo "net.ipv4.conf.all.forwarding = 1" >> /etc/sysctl.conf
      # sysctl -p
      SHELL
  end

  # \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\
  config.vm.define "Debian12-WebServer" do |webserver|
  webserver.vm.box = "/home/max/vagrant/images/debian12"
  webserver.vm.network :private_network,
       :type => 'ip',
       :libvirt__forward_mode => 'veryisolated',
       :libvirt__dhcp_enabled => false,
       :ip => '192.168.1.2',
       :libvirt__netmask => '255.255.255.248',
       :libvirt__network_name => 'vagrant-libvirt-inet1',
       :libvirt__always_destroy => false
  webserver.vm.provider "libvirt" do |lvirt|
      lvirt.memory = "1024"
      lvirt.cpus = "1"
      lvirt.title = "Debian12-WebServer"
      lvirt.description = "Виртуальная машина на базе дистрибутива Debian Linux. WebServer"
      lvirt.management_network_name = "vagrant-libvirt-mgmt"
      lvirt.management_network_address = "192.168.121.0/24"
      lvirt.management_network_keep = "true"
      lvirt.management_network_mac = "52:54:00:27:28:84"
  #   lv.storage :file, :size => '1G', :device => 'vdb', :allow_existing => false
  end
  webserver.vm.provision "file", source: "logs/nginx.conf", destination: "~/nginx.conf"
  # webserver.vm.provision "file", source: "logs/audit.rules", destination: "~/audit.rules"
  webserver.vm.provision "shell", inline: <<-SHELL
      brd='*************************************************************'
      echo "$brd"
      echo 'Set Hostname'
      hostnamectl set-hostname webserver
      echo "$brd"
      sed -i 's/debian12/webserver/' /etc/hosts
      sed -i 's/debian12/webserver/' /etc/hosts
      echo "$brd"
      echo 'Изменим ttl для работы через раздающий телефон'
      echo "$brd"
      sysctl -w net.ipv4.ip_default_ttl=66
      echo "$brd"
      echo 'Если ранее не были установлены, то установим необходимые  пакеты'
      echo "$brd"
      apt update
      export DEBIAN_FRONTEND=noninteractive
      apt install -y iptables nginx rsyslog auditd audispd-plugins
      # Настраиваем nginx на отправку журналов в rsyslog.
      cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.default
      cp /home/vagrant/nginx.conf /etc/nginx/nginx.conf
      systemctl restart nginx.service
      # Включаем отправку сообщений от службы auditd на удаленный хост.
      sed -i 's/active = no/active = yes/' /etc/audit/plugins.d/au-remote.conf
      sed -i 's/remote_server = /remote_server = 192.168.1.1/' /etc/audit/audisp-remote.conf
      # Добавляем правило для мониторинга изменений в конфиге nginx.conf со стороны auditd.
      echo '-w /etc/nginx/nginx.conf -p wa' >> /etc/audit/rules.d/10-audit.rules
      # Загружаем новое правило.
      augenrules --load
      echo '' >> /etc/rsyslog.conf
      echo 'security.* @@192.168.1.1:514' >> /etc/rsyslog.conf
      systemctl restart rsyslog.service
      systemctl restart auditd.service
      SHELL
  end
  # \\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\
  config.vm.define "Debian12-AppServer" do |appserver|
  appserver.vm.box = "/home/max/vagrant/images/debian12"
  appserver.vm.network :private_network, 
       :type => 'ip',
       :libvirt__forward_mode => 'veryisolated',
       :libvirt__dhcp_enabled => false,  
       :ip => '192.168.1.3',
       :libvirt__netmask => '255.255.255.248',
       :libvirt__network_name => 'vagrant-libvirt-inet1',
       :libvirt__always_destroy => false    
  appserver.vm.provider "libvirt" do |lvirt|
      lvirt.memory = "1024"
      lvirt.cpus = "1"
      lvirt.title = "Debian12-AppServer"    
      lvirt.description = "Виртуальная машина на базе дистрибутива Debian Linux. AppServer"
      lvirt.management_network_name = "vagrant-libvirt-mgmt"
      lvirt.management_network_address = "192.168.121.0/24"
      lvirt.management_network_keep = "true"
      lvirt.management_network_mac = "52:54:00:27:28:85"
  #   lv.storage :file, :size => '1G', :device => 'vdb', :allow_existing => false
  end
  appserver.vm.provision "file", source: "logs/nginx.conf", destination: "~/nginx.conf"
  appserver.vm.provision "shell", inline: <<-SHELL
      brd='*************************************************************'
      echo "$brd"
      echo 'Set Hostname'
      hostnamectl set-hostname appserver
      echo "$brd"
      sed -i 's/debian12/appserver/' /etc/hosts
      sed -i 's/debian12/appserver/' /etc/hosts
      echo "$brd"
      echo 'Изменим ttl для работы через раздающий телефон'
      echo "$brd"
      sysctl -w net.ipv4.ip_default_ttl=66
      echo "$brd"
      echo 'Если ранее не были установлены, то установим необходимые  пакеты'
      echo "$brd"
      apt update
      export DEBIAN_FRONTEND=noninteractive
      apt install -y iptables rsyslog
      echo '' >> /etc/rsyslog.conf
      echo '*.* @@192.168.1.1:514' >> /etc/rsyslog.conf
      systemctl restart rsyslog.service
      SHELL
  end
end
