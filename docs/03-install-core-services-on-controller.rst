.. highlight:: none

3. Install core services on controller
======================================

This page is based on the following OpenStack Installation Guide pages:

http://docs.openstack.org/liberty/install-guide-rdo/environment-sql-database.html

http://docs.openstack.org/liberty/install-guide-rdo/environment-nosql-database.html

http://docs.openstack.org/liberty/install-guide-rdo/environment-messaging.html

1. Install MariaDB::

    # yum install mariadb mariadb-server MySQL-python
2. Set some needed MariaDB configuration parameters::

    # vim /etc/my.cnf

      bind-address = 172.16.0.192
      default-storage-engine = innodb
      innodb_file_per_table
      collation-server = utf8_general_ci
      init-connect = 'SET NAMES utf8'
      character-set-server = utf8
3. Enable and start the MariaDB service::

    # systemctl enable mariadb.service
    # systemctl start mariadb.service
4. Initialize MariaDB security. Say 'yes' to all prompts, and set a good root password::

    # mysql_secure_installation
5. Set up the MySQL client configuration::

    # vim /root/.my.cnf

      [client]
      user=root
      password=*MYSQLROOTPASSWORD*
6. Install MongoDB::

    # yum install mongodb-server mongodb
7. Set some needed MongoDB configuration parameters::

    # vim /etc/mongod.conf

      bind_ip = 172.16.0.192
      smallfiles = true
8. Enable and start the MongoDB service::

    # systemctl enable mongod.service
    # systemctl start mongod.service
9. Install RabbitMQ::

     # yum install rabbitmq-server
10. Enable and start the RabbitMQ service::

     # systemctl enable rabbitmq-server.service
     # systemctl start rabbitmq-server.service
11. Create the "openstack" RabbitMQ user::

     # rabbitmqctl add_user openstack *RABBIT_PASS*
     # rabbitmqctl set_permissions openstack ".*" ".*" ".*"
