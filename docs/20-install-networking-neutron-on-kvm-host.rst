.. highlight:: none

20. Install Networking (neutron) on KVM Host
============================================

This page is based on the following OpenStack Installation Guide pages:

http://docs.openstack.org/liberty/install-guide-rdo/neutron-compute-install.html

**All steps except 2 have modifications for XenServer.**

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
      lock_path = /var/lib/neutron/tmp
* Make sure that any connection options under ``[database]`` are deleted or commented-out.

* Delete or comment-out any pre-existing lines in the ``[keystone_authtoken]`` section.

3. **Configure the neutron ovs agent. Replace** ``*XAPI_BRIDGE*`` **with your own**::

    # vim /etc/neutron/plugins/ml2/openvswitch_agent.ini

      [ovs]
      integration_bridge = *XAPI_BRIDGE*
      bridge_mappings = public:br-eth0

      [securitygroup]
      firewall_driver = neutron.agent.firewall.NoopFirewallDriver

4. **Reconfigure nova to use neutron. Replace** ``*NEUTRON_PASS*`` **and** ``*XAPI_BRIDGE*`` **with your own**::

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
      ovs_bridge = *XAPI_BRIDGE*

      [DEFAULT]
      linuxnet_ovs_integration_bridge = *XAPI_BRIDGE*

5. **Enable and start the ovs service**::

    # systemctl enable openvswitch.service
    # systemctl start openvswitch.service
6. **Set up the ovs bridge to the public network**::

    # ovs-vsctl add-br br-eth0
    # ovs-vsctl add-port br-eth0 eth0
7. **Enable and start the neutron service**::

    # systemctl enable neutron-openvswitch-agent.service
    # systemctl start neutron-openvswitch-agent.service
