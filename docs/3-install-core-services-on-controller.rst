.. highlight:: none

3. Install core services on controller
======================================

1. # yum install mariadb mariadb-server MySQL-python
2. # vim /etc/my.cnf
  bind-address = 172.16.0.192
  default-storage-engine = innodb
  innodb_file_per_table
  collation-server = utf8_general_ci
  init-connect = 'SET NAMES utf8'
  character-set-server = utf8
3. # systemctl enable mariadb.service
4. # systemctl start mariadb.service
5. # mysql_secure_installation
  • Say yes to everything and set a good root password
6. # vim /root/.my.cnf
  [client]
  user=root
  password=*MYSQLROOTPASSWORD*

7. # yum install mongodb-server mongodb
8. # vim /etc/mongod.conf
  bind_ip = 172.16.0.192
  smallfiles = true
9. # systemctl enable mongod.service
10. # systemctl start mongod.service

11. # yum install rabbitmq-server
12. # systemctl enable rabbitmq-server.service
13. # systemctl start rabbitmq-server.service
14. # rabbitmqctl add_user openstack *RABBIT_PASS*
# rabbitmqctl set_permissions openstack ".*" ".*" ".*"
