.. highlight:: none

11. Install Networking (neutron) on compute VM
==============================================

This page is based on the following OpenStack Installation Guide page:

http://docs.openstack.org/liberty/install-guide-rdo/neutron-compute-install.html

It is also based on some steps from the following guide:

https://www.citrix.com/blogs/2015/11/30/integrating-xenserver-rdo-and-neutron/

**Steps 1, 3, 4, 6, and 8 have specific changes for the use of XenServer.**

1. **Install the neutron and ovs packages**::

    # yum install openstack-neutron openstack-neutron-openvswitch ebtables ipset openvswitch
2. Configure neutron. Replace ``*RABBIT_PASS*`` and ``*NEUTRON_PASS*`` with your own::

    # vim /etc/neutron/neutron.conf

      [DEFAULT]
      rpc_backend = rabbit
      auth_strategy = keystone

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

      [oslo_concurrency]
      Lock_path = /var/lib/neutron/tmp
* Make sure that any connection options under ``[database]`` are deleted or commented-out.

* Delete or comment-out any pre-existing lines in the ``[keystone_authtoken]`` section.

3. **Configure the neutron ovs agent. Replace** ``*XAPI_BRIDGE*`` **and** ``*XENSERVER_ROOT*`` **with your own**::

    # vim /etc/neutron/plugins/ml2/openvswitch_agent.ini

      [ovs]
      integration_bridge = *XAPI_BRIDGE*
      bridge_mappings = public:xenbr0

      [agent]
      root_helper = neutron-rootwrap-xen-dom0 /etc/neutron/rootwrap.conf
      root_helper_daemon =
      minimize_polling = False

      [securitygroup]
      firewall_driver = neutron.agent.firewall.NoopFirewallDriver
4. **Configure neutron rootwrap to connect to XenServer. Replace** ``*XENSERVER_ROOT*`` **with your own**::

    # vim /etc/neutron/rootwrap.conf

      [xenapi]
      xenapi_connection_url=http://compute1
      xenapi_connection_username=root
      xenapi_connection_password=*XENSERVER_ROOT*
* There are other lines already present in this file. These should be left as-is.
5. Reconfigure nova to use neutron. Replace ``*NEUTRON_PASS*`` with your own::

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

6. **Use the helper script to install the dom0 neutron plugins**::

    # source rdo_xenserver_helper.sh
    # install_dom0_plugins
* Enter the XenServer root password when prompted (twice).

7. Restart the nova service::

    # systemctl restart openstack-nova-compute.service
8. **Enable and start the neutron service**::

    # systemctl enable neutron-openvswitch-agent.service
    # systemctl start neutron-openvswitch-agent.service
