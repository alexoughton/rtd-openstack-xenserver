.. highlight:: none

6. Install Compute (nova) on controller
========================================

This page is based on the following OpenStack Installation Guide pages:

http://docs.openstack.org/liberty/install-guide-rdo/nova-controller-install.html

1. Open the MySQL client and create the "nova" database. Replace ``*NOVA_DBPASS*`` with your own::

    # mysql

      > create database nova;
      > grant all privileges on nova.* to 'nova'@'localhost' identified by '*NOVA_DBPASS*';
      > grant all privileges on nova.* to 'nova'@'%' identified by '*NOVA_DBPASS*';
      > quit
2. Create the "nova" user, role, service and endpoints. Provide ``*NOVA_PASS*`` when prompted::

    # source admin-openrc.sh
    # openstack user create --domain default --password-prompt nova
    # openstack role add --project service --user nova admin
    # openstack service create --name nova --description "OpenStack Compute" compute
    # openstack endpoint create --region RegionOne compute public http://controller:8774/v2/%\(tenant_id\)s
    # openstack endpoint create --region RegionOne compute internal http://controller:8774/v2/%\(tenant_id\)s
    # openstack endpoint create --region RegionOne compute admin http://controller:8774/v2/%\(tenant_id\)s
3. Install nova packages::

    # yum install openstack-nova-api openstack-nova-cert openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy openstack-nova-scheduler   python-novaclient
4. Configure nova. Replace ``*NOVA_DBPASS*``, ``*NOVA_PASS*``, ``*SERVER_IP*`` and ``*RABIT_PASS*`` with your own::

    # vim /etc/nova/nova.conf

    [database]
    connection = mysql://nova:*NOVA_DBPASS*@controller/nova

    [DEFAULT]
    rpc_backend = rabbit
    auth_strategy = keystone
    my_ip = *SERVER_IP*
    network_api_class = nova.network.neutronv2.api.API
    security_group_api = neutron
    linuxnet_interface_driver = nova.network.linux_net.NeutronLinuxBridgeInterfaceDriver
    firewall_driver = nova.virt.firewall.NoopFirewallDriver
    enabled_apis = osapi_compute,metadata

    [oslo_messaging_rabbit]
    rabbit_host = controller
    rabbit_userid = openstack
    rabbit_password = *RABBIT_PASS*

    [keystone_authtoken]
    auth_uri = http://controller:5000
    auth_url = http://controller:35357
    auth_plugin = password
    project_domain_id = default
    user_domain_id = default
    project_name = service
    username = nova
    password = *NOVA_PASS*

    [vnc]
    vncserver_listen = $my_ip
    vncserver_proxyclient_address = $my_ip

    [glance]
    host = controller

    [oslo_concurrency]
    lock_path = /var/lib/nova/tmp
5. Populate the nova database::

    # su -s /bin/sh -c "nova-manage db sync" nova
6. Enable and start the nova service::

    # systemctl enable openstack-nova-api.service openstack-nova-cert.service openstack-nova-consoleauth.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service
    # systemctl start openstack-nova-api.service openstack-nova-cert.service openstack-nova-consoleauth.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service
