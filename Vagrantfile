# -*- mode: ruby -*-
# vi: set ft=ruby :
# Установка переменной окружения, задающей российское зеркало для box-файлов
ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'

Vagrant.configure("2") do |config|
  # Создание виртуальной машины веб-сервера
  config.vm.define "webserver" do |web|
    web.vm.box = "bento/ubuntu-24.04"
    web.vm.network "private_network", ip: "192.168.50.10"
    
    web.vm.provider "virtualbox" do |w_vb|
      w_vb.memory = 4048
      w_vb.cpus = 4
    end
    
    # Установка Apache
    web.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install apache2 -y
      sudo systemctl enable apache2
      sudo apt-get install -y mysql-client
      sudo apt install php libapache2-mod-php php-mysql -y
      sudo systemctl restart apache2
      cat <<EOF | sudo tee /var/www/html/test_db.php
<?php
\$host = "192.168.50.11";
\$user = "vagrant_test";
\$pass = "Tusur123";
\$db = "testdb";
\$conn = new mysqli(\$host, \$user, \$pass, \$db);
if (\$conn->connect_error) {
 die("Connection failed: " . \$conn->connect_error);
}
echo "Connected to MySQL successfully!";
?>
EOF
    SHELL
  end

  # Создание виртуальной машины базы данных
  config.vm.define "dbserver" do |db|
    db.vm.box = "bento/ubuntu-24.04"
    db.vm.network "private_network", ip: "192.168.50.11"
    
    db.vm.provider "virtualbox" do |db_vb|
      db_vb.memory = 4048
      db_vb.cpus = 4
    end

    # Установка MySQL
    db.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y mysql-server
      sudo systemctl enable mysql
      sudo mysql -u root <<MYSQL_SCRIPT
        CREATE DATABASE testdb;
        USE testdb;
        CREATE TABLE users (id INT, name VARCHAR(20));
        INSERT INTO users VALUES (1, 'vagrant_test');
        CREATE USER 'vagrant_test'@'192.168.50.10' IDENTIFIED BY 'Tusur123';
        GRANT ALL PRIVILEGES ON testdb.* TO 'vagrant_test'@'192.168.50.10';
        FLUSH PRIVILEGES;
MYSQL_SCRIPT
      sudo sed -i 's/bind-address = 127.0.0.1/bind-address = 192.168.50.11/' /etc/mysql/mysql.conf.d/mysqld.cnf
      sudo systemctl restart mysql
    SHELL
  end

  # Создание виртуальной машины балансировщика нагрузки
  config.vm.define "loadbalancer" do |lb|
    lb.vm.box = "bento/ubuntu-24.04"
    lb.vm.network "public_network"
    
    lb.vm.provider "virtualbox" do |lb_vb|
      lb_vb.memory = 4048
      lb_vb.cpus = 4
    end
    
    # Установка HAProxy
    lb.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y haproxy
      sudo systemctl enable haproxy
      cat <<EOF | sudo tee -a /etc/haproxy/haproxy.cfg

frontend http_front
  bind *:80
  default_backend http_back

backend http_back
  balance roundrobin
  server webserver1 192.168.50.10:80 check inter 5000 rise 2 fall 3
EOF
      sudo systemctl restart haproxy
    SHELL
  end

  config.vm.boot_timeout = 600  # Увеличиваем таймаут
end