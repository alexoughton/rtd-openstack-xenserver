.. highlight:: none

11. Install Networking (neutron) on compute VM
==============================================

This page is based on the following OpenStack Installation Guide pages:

http://docs.openstack.org/liberty/install-guide-rdo/neutron-compute-install.html

http://docs.openstack.org/liberty/install-guide-rdo/launch-instance.html

http://docs.openstack.org/liberty/install-guide-rdo/launch-instance-networks-public.html

It is also based on some steps from the following guide:

https://www.citrix.com/blogs/2015/11/30/integrating-xenserver-rdo-and-neutron/

**Steps 1, 3, 4, 6, 8, 11 and 14 have specific changes for the use of XenServer.**

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
9. Log on to the controller node as root.
10. Load the "admin" credential file::

    # source admin-openrc.sh
11. **Check the neutron agent list**::

     # neutron agent-list

       +--------------------------------------+--------------------+---------------------------------------------+-------+----------------+---------------------------+
       | id                                   | agent_type         | host                                        | alive | admin_state_up | binary                    |
       +--------------------------------------+--------------------+---------------------------------------------+-------+----------------+---------------------------+
       | 57c49643-3e48-4252-9665-2f22e3b93b0e | Open vSwitch agent | compute1-vm.openstack.lab.eco.rackspace.com | :-)   | True           | neutron-openvswitch-agent |
       | 977ff9ae-96e5-4ef9-93d5-65a8541d7d25 | Metadata agent     | controller.openstack.lab.eco.rackspace.com  | :-)   | True           | neutron-metadata-agent    |
       | ca0fb18a-b3aa-4cd1-bc5f-ba4700b4d9ce | Open vSwitch agent | controller.openstack.lab.eco.rackspace.com  | :-)   | True           | neutron-openvswitch-agent |
       | d42db23f-3738-48b3-8f83-279ee29e84ef | DHCP agent         | controller.openstack.lab.eco.rackspace.com  | :-)   | True           | neutron-dhcp-agent        |
       +--------------------------------------+--------------------+---------------------------------------------+-------+----------------+---------------------------+
* The list should include the ovs agent running on ``controller`` and ``compute1-vm``.

12. Create the default security group::

     # nova secgroup-add-rule default icmp -1 -1 0.0.0.0/0
     # nova secgroup-add-rule default tcp 1 65535 0.0.0.0/0
13. Create the public network. Replace ``*PUBLIC_NETWORK_CIDR*``, ``*START_IP_ADDRESS*``, ``*END_IP_ADDRESS*`` ``*DNS_RESOLVER*`` and ``*PUBLIC_NETWORK_GATEWAY*`` with your own::

     # neutron net-create public --shared --provider:physical_network public --provider:network_type flat
     # neutron subnet-create public *PUBLIC_NETWORK_CIDR* --name public --allocation-pool start=*START_IP_ADDRESS*,end=*END_IP_ADDRESS* --dns-nameserver *DNS_RESOLVER* --gateway *PUBLIC_NETWORK_GATEWAY*

14. **There is a bug regarding the network's segmentation ID which needs to be fixed. This should be resolved in openstack-neutron-7.0.1, but if you are running an older version**:

     a. Update the `segmentation_id` field in the `neutron` database::

         # mysql neutron
           > update ml2_network_segments set segmentation_id=0;
           > quit
     b. Update the segmentation_id for the DHCP agent's ovs port::

         # ovs-vsctl set Port $(ovs-vsctl show | grep Port | grep tap | awk -F \" ' { print $2 } ') other_config:segmentation_id=0
