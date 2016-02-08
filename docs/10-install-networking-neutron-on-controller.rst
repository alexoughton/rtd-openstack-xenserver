.. highlight:: none

10. Install Networking (neutron) on controller
==============================================

This page is based on the following OpenStack Installation Guide page:

http://docs.openstack.org/liberty/install-guide-rdo/neutron-controller-install.html

**Steps 3, 5, 6, 7, 9, 12, 13 and 15 have specific changes for the use of XenServer.**

1. Open the MySQL client and create the "glance" database. Replace ``*NEUTRON_DBPASS*`` with your own::

    # mysql
      > create database neutron;
      > grant all privileges on neutron.* to 'neutron'@'localhost' identified by '*NEUTRON_DBPASS*';
      > grant all privileges on neutron.* to 'neutron'@'%' identified by '*NEUTRON_DBPASS*';
      > quit
2. Create the "neutron" user, role, service and endpoints. Provide ``*NEUTRON_PASS*`` when prompted::

    # source admin-openrc.sh
    # openstack user create --domain default --password-prompt neutron
    # openstack role add --project service --user neutron admin
    # openstack service create --name neutron --description "OpenStack Networking" network
    # openstack endpoint create --region RegionOne network public http://controller:9696
    # openstack endpoint create --region RegionOne network internal http://controller:9696
    # openstack endpoint create --region RegionOne network admin http://controller:9696
3. **Install the neutron and ovs packages**::

    # yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-openvswitch python-neutronclient ebtables ipset
4. Configure neutron. Note that the default file already has lines for keystone_authtoken. These must be deleted. Replace ``*NEUTRON_DBPASS*``, ``*NEUTRON_PASS*``, ``*RABBIT_PASS*`` and ``*NOVA_PASS*`` with your own::

    # vim /etc/neutron/neutron.conf

      [database]
      connection = mysql://neutron:*NEUTRON_DBPASS*@controller/neutron
      rpc_backend = rabbit

      [DEFAULT]
      core_plugin = ml2
      service_plugins =
      auth_strategy = keystone
      notify_nova_on_port_status_changes = True
      notify_nova_on_port_data_changes = True
      nova_url = http://controller:8774/v2

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
      username = neutron
      password = *NEUTRON_PASS*

      [nova]
      auth_url = http://controller:35357
      auth_plugin = password
      project_domain_id = default
      user_domain_id = default
      region_name = RegionOne
      project_name = service
      username = nova
      password = *NOVA_PASS*

      [oslo_concurrency]
      lock_path = /var/lib/neutron/tmp
5. **Configure the ml2 plugin**::

    # vim /etc/neutron/plugins/ml2/ml2_conf.ini

      [ml2]
      type_drivers = flat,vlan
      tenant_network_types =
      mechanism_drivers = openvswitch
      extension_drivers = port_security

      [ml2_type_flat]
      flat_networks = public

      [securitygroup]
      Enable_ipset = True

6. **Configure ml2's ovs plugin. Replace **``*XAPI_BRIDGE*`` **with your own**::

    # vim /etc/neutron/plugins/ml2/openvswitch_agent.ini

      [ovs]
      integration_driver = *XAPI_BRIDGE*
      bridge_mappings = public:br-eth0

      [securitygroup]
      Firewall_driver = neutron.agent.firewall.NoopFirewallDriver

7. **Configure the DHCP Agent. Replace **``*XAPI_BRIDGE*`` **with your own**::

    # vim /etc/neutron/dhcp_agent.ini

      [DEFAULT]
      interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
      ovs_integration_bridge = *XAPI_BRIDGE*
      dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
      enable_isolated_metadata= True
8. Configure the metadata agent. Note that the default file already has some lines in [DEFAULT]. These need to be commented-out or deleted. Replace ``*NEUTRON_PASS*`` and ``*NEUTRON_METADATA_SECRET*`` with your own::

    # vim /etc/neutron/metadata_agent.ini

      [DEFAULT]
      auth_uri = http://controller:5000
      auth_url = http://controller:35357
      auth_region = RegionOne
      auth_plugin = password
      project_domain_id = default
      user_domain_id = default
      project_name = service
      username = neutron
      password = *NEUTRON_PASS*
      nova_metadata_ip = controller
      metadata_proxy_shared_secret = *NEUTRON_METADATA_SECRET*
9. **Reconfigure nova to use neutron. Replace **``*NEUTRON_PASS*``**,** ``*NEUTRON_METADATA_SECRET*`` **and** ``*XAPI_BRIDGE*`` **with your own**::

    # vim /etc/nova/nova.conf

      [neutron]
      url = http://controller:9696
      auth_url = http://controller:35357
      auth_plugin = password
      project_domain_id = default
      user_domain_id = default
      region_name = RegionOne
      project_name = service
      username = neutron
      password = *NEUTRON_PASS*
      service_metadata_proxy = True
      metadata_proxy_shared_secret = *NEUTRON_METADATA_SECRET*
      ovs_bridge = *XAPI_BRIDGE*

10. Symlink the ml2 configuration file to neutron's plugin.ini file::

     # ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
11. Populate the neutron database::

     # su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf -config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
12. **Enable and start the ovs service**::

     # systemctl enable openvswitch.service
     # systemctl start openvswitch.service
13. **Set up the ovs bridge to the public network**::

     # ovs-vsctl add-br br-eth0
     # ovs-vsctl add-port br-eth0 eth0
14. Restart the nova service::

     # systemctl restart openstack-nova-api.service
15. **Enable and start the neutron services**::

     # systemctl enable neutron-server.service neutron-openvswitch-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service neutron-ovs-cleanup.service
     # systemctl start neutron-server.service neutron-openvswitch-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service neutron-ovs-cleanup.service
