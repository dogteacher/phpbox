# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|

  config.vm.box = "chef/centos-6.5"
  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder "../", "/var/www/html"

  config.vm.provision "shell", inline: <<-SHELL

DBPASS=vagrant

yum -y update
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
rpm -Uvh epel-release-6-8.noarch.rpm
rpm -Uvh rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
rpm -Uvh remi-release-6.rpm
sed -i -e 's/enabled *= *1/enabled=0/g' /etc/yum.repos.d/epel.repo
sed -i -e 's/enabled *= *1/enabled=0/g' /etc/yum.repos.d/rpmforge.repo
sed -i -e 's/enabled *= *1/enabled=0/g' /etc/yum.repos.d/remi.repo


#
# apache 2.2.15
#
yum -y install httpd
chkconfig httpd on
service iptables stop
chkconfig iptables off

grep '^LoadModule php5_module' /etc/httpd/conf/httpd.conf
if [ $? -eq 1 ]; then
cat >> /etc/httpd/conf/httpd.conf << "EOF"
LoadModule php5_module modules/libphp5.so
<FilesMatch "\.php$">
  SetHandler application/x-httpd-php
</FilesMatch>
EOF
fi


#
# php 5.6.5
#
yum -y install php php-mbstring php-pear php-gd
yum --enablerepo=remi,epel,rpmforge,remi-php56 -y install php php-mbstring php-pear

grep '^date.timezone = Asia/Tokyo' /etc/php.ini
if [ $? -eq 1 ]; then
cat >> /etc/php.ini << "EOF"
date.timezone = Asia/Tokyo
mbstring.language = Japanese
default_charset = UTF-8
mbstring.detect_order = UTF-8,EUC-JP,SJIS,JIS,ASCII
EOF
fi


#
# mysql 5.6.23
#
wget http://dev.mysql.com/get/mysql-community-release-el6-5.noarch.rpm
rpm -Uvh mysql-community-release-el6-5.noarch.rpm
yum -y install mysql-community-server
chkconfig mysqld on

grep '^character-set-server = utf8' /etc/my.cnf
if [ $? -eq 1 ]; then
service mysqld start
mysql -u root -e "delete from mysql.user where host <> 'localhost' or user <> 'root';"
mysql -u root -e "SET PASSWORD FOR root@localhost=PASSWORD('$DBPASS');"
service mysqld stop

sed -i '/mysqld\]/a character-set-server = utf8' /etc/my.cnf
cat >> /etc/my.cnf << "EOF"
[client]
default-character-set = utf8
[mysql]
default-character-set = utf8
[mysqldump]
default-character-set = utf8
EOF
fi

#
# phpmyadmin 4.3.9
#
yum --enablerepo=remi,epel,rpmforge,remi-php56 -y install phpMyAdmin php-mysql php-mcrypt
sed -i 's/Allow from 127.0.0.1/Allow from 192.168/g' /etc/httpd/conf.d/phpMyAdmin.conf


service httpd start
service mysqld start
  SHELL
end

