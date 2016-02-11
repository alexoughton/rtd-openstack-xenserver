.. highlight:: none

4. Install Identity (keystone) on controller
============================================

This page is based on the following OpenStack Installation Guide pages:

http://docs.openstack.org/liberty/install-guide-rdo/keystone-install.html

http://docs.openstack.org/liberty/install-guide-rdo/keystone-services.html

http://docs.openstack.org/liberty/install-guide-rdo/keystone-users.html

http://docs.openstack.org/liberty/install-guide-rdo/keystone-verify.html

http://docs.openstack.org/liberty/install-guide-rdo/keystone-openrc.html

1. Open the MySQL client and create the "keystone" database. Replace ``*KEYSTONE_DBPASS*`` with your own::

    # mysql
      > create database keystone;
      > grant all privileges on keystone.* to 'keystone'@'localhost' identified by '*KEYSTONE_DBPASS*';
      > grant all privileges on keystone.* to 'keystone'@'%' identified by '*KEYSTONE_DBPASS*';
      > quit
2. Install the keystone packages::

    # yum install openstack-keystone httpd mod_wsgi  memcached python-memcached
3. Enable and start the memcached service::

    # systemctl enable memcached.service
    # systemctl start memcached.service
4. Configure keystone. Replace ``*ADMIN_TOKEN*`` and ``*KEYSTONE_DBPASS*`` with your own::

    # vim /etc/keystone/keystone.conf

      [DEFAULT]
      admin_token = *ADMIN_TOKEN*

      [database]
      connection = mysql://keystone:*KEYSTONE_DBPASS*@controller/keystone

      [memcache]
      servers = localhost:11211

      [token]
      provider = uuid
      driver = memcache

      [revoke]
      driver = sql
5. Populate the keystone database::

    # su -s /bin/sh -c "keystone-manage db_sync" keystone

6. Set the Apache server name::

    # vim /etc/httpd/conf/httpd.conf

      ServerName controller

7. Configure wsgi::

     # vim /etc/httpd/conf.d/wsgi-keystone.conf

       Listen 5000
       Listen 35357

       <VirtualHost *:5000>
           WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
           WSGIProcessGroup keystone-public
           WSGIScriptAlias / /usr/bin/keystone-wsgi-public
           WSGIApplicationGroup %{GLOBAL}
           WSGIPassAuthorization On
           <IfVersion >= 2.4>
             ErrorLogFormat "%{cu}t %M"
           </IfVersion>
           ErrorLog /var/log/httpd/keystone-error.log
           CustomLog /var/log/httpd/keystone-access.log combined

           <Directory /usr/bin>
               <IfVersion >= 2.4>
                   Require all granted
               </IfVersion>
               <IfVersion < 2.4>
                   Order allow,deny
                   Allow from all
               </IfVersion>
           </Directory>
       </VirtualHost>

       <VirtualHost *:35357>
           WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
           WSGIProcessGroup keystone-admin
           WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
           WSGIApplicationGroup %{GLOBAL}
           WSGIPassAuthorization On
           <IfVersion >= 2.4>
             ErrorLogFormat "%{cu}t %M"
           </IfVersion>
           ErrorLog /var/log/httpd/keystone-error.log
           CustomLog /var/log/httpd/keystone-access.log combined

           <Directory /usr/bin>
               <IfVersion >= 2.4>
                   Require all granted
               </IfVersion>
               <IfVersion < 2.4>
                   Order allow,deny
                   Allow from all
               </IfVersion>
           </Directory>
       </VirtualHost>
8. Enable and start the Apache service::

    # systemctl enable httpd.service
    # systemctl start httpd.service
9. Set up temportary connection parameters. Replace ``*ADMIN_TOKEN*`` with your own::

    # export OS_TOKEN=*ADMIN_TOKEN*
    # export OS_URL=http://controller:35357/v3
    # export OS_IDENTITY_API_VERSION=3
10. Create keystone service and endpoints::

    # openstack service create --name keystone --description "OpenStack Identity" identity
    # openstack endpoint create --region RegionOne identity public http://controller:5000/v2.0
    # openstack endpoint create --region RegionOne identity internal http://controller:5000/v2.0
    # openstack endpoint create --region RegionOne identity admin http://controller:35357/v2.0

11. Create the "admin" project, user and role. Provide your ``*ADMIN_PASS*`` twice when prompted::

     # openstack project create --domain default --description "Admin Project" admin
     # openstack user create --domain default --password-prompt admin
     # openstack role create admin
     # openstack role add --project admin --user admin admin

12. Create the "service" project::

     # openstack project create --domain default --description "Service Project" service
13. Create the "demo" project, user and role. Provide your ``*DEMO_PASS*`` twice when prompted::

     # openstack project create --domain default --description "Demo Project" demo
     # openstack user create --domain default --password-prompt demo
     # openstack role create user
     # openstack role add --project demo --user demo user

14. Disable authentication with the admin token::

     # vim /usr/share/keystone/keystone-dist-paste.ini
* Remove ``admin_token_auth`` from ``[pipeline:public_api]``, ``[pipeline:admin_api]`` and ``[pipeline:api_v3]``

15. Disable the temporary connection parameters::

     # unset OS_TOKEN OS_URL
16. Test authentication for the "admin" user. Provide ``*ADMIN_PASS*`` when prompted::

     # openstack --os-auth-url http://controller:35357/v3 --os-project-domain-id default --os-user-domain-id default --os-project-name admin --os-username admin --os-auth-type password token issue
* If this is working, various values will be returned (yours will be different)::

    +------------+----------------------------------+
    | Field      | Value                            |
    +------------+----------------------------------+
    | expires    | 2016-02-05T22:55:18.580385Z      |
    | id         | 9bd8b09e4fdd43cea1f32ca6d62c946b |
    | project_id | 76f8c8fd7b1e407d97c4604eb2a408b3 |
    | user_id    | 31766cbe74d541088c6ba2fd24654034 |
    +------------+----------------------------------+

17. Test authentication for the "demo" user. Provide \*DEMO_PASS\ when prompted::

     # openstack --os-auth-url http://controller:5000/v3 --os-project-domain-id default --os-user-domain-id default --os-project-name demo --os-username demo --os-auth-type password token issue
* Again, if this is working, various values will be returned.

18. Create permanent client authentication file for the "admin" user. Replace ``*ADMIN_PASS*`` with your own::

     # vim /root/admin-openrc.sh

       export OS_PROJECT_DOMAIN_ID=default
       export OS_USER_DOMAIN_ID=default
       export OS_PROJECT_NAME=admin
       export OS_TENANT_NAME=admin
       export OS_USERNAME=admin
       export OS_PASSWORD=*ADMIN_PASS*
       export OS_AUTH_URL=http://controller:35357/v3
       export OS_IDENTITY_API_VERSION=3
19. Create permanent client authentication file for the "demo" user. Replace ``*DEMO_PASS*`` with your own::

     # vim /root/demo-openrc.sh

       export OS_PROJECT_DOMAIN_ID=default
       export OS_USER_DOMAIN_ID=default
       export OS_PROJECT_NAME=demo
       export OS_TENANT_NAME=demo
       export OS_USERNAME=demo
       export OS_PASSWORD=*DEMO_PASS*
       export OS_AUTH_URL=http://controller:5000/v3
       export OS_IDENTITY_API_VERSION=3
20. Test authentication with the permanent settings::

     # source admin-openrc.sh
     # openstack token issue
* Once more, if this works, various values will be returned.
