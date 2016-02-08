.. highlight:: none

9. Install Compute (nova) on XenServer compute VM
=================================================

This page is based on the following OpenStack Installation Guide pages:

http://docs.openstack.org/liberty/install-guide-rdo/nova-compute-install.html

http://docs.openstack.org/liberty/install-guide-rdo/nova-verify.html

It is also based on some steps from the following guide:

https://www.citrix.com/blogs/2015/11/30/integrating-xenserver-rdo-and-neutron/

**All steps have modifications for XenServer.**

1. Download and install pip, and xenapi::

    # wget https://bootstrap.pypa.io/get-pip.py
    # python get-pip.py
    # pip install xenapi
2. Install nova packages::

    # yum install openstack-nova-compute sysfsutils
3. Configure nova. Replace ``*XENSERVER_ROOT*``, ``*CONTROLLER_ADDRESS*``, ``*XAPI_BRIDGE*``, ``*VM_IP*``, ``*NOVA_PASS*``, ``*XENSERVER_IP*`` and ``*RABIT_PASS*`` with your own::

    # vim /etc/nova/nova.conf

      [DEFAULT]
      rpc_backend = rabbit
      auth_strategy = keystone
      my_ip = *VM_IP*
      network_api_class = nova.network.neutronv2.api.API
      security_group_api = neutron
      linuxnet_interface_driver = nova.network.linux_net.NeutronLinuxBridgeInterfaceDriver
      firewall_driver = nova.virt.firewall.NoopFirewallDriver
      compute_driver = xenapi.XenAPIDriver

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
      enabled = True
      vncserver_listen = 0.0.0.0
      vncserver_proxyclient_address = *IP ADDRESS OF XENSERVER*
      novncproxy_base_url = http://*CONTROLLER_ADDRESS*:6080/vnc_auto.html

      [glance]
      host = controller

      [oslo_concurrency]
      lock_path = /var/lib/nova/tmp

      [xenserver]
      connection_url=http://compute1
      connection_username=root
      connection_password=*XENSERVER_ROOT*
      vif_driver=nova.virt.xenapi.vif.XenAPIOpenVswitchDriver
      ovs_int_bridge=*XAPI_BRIDGE*
      ovs_integration_bridge=*XAPI_BRIDGE*
4. Download and modify a helper script for installing the dom0 plugins::

    # wget --no-check-certificate https://raw.githubusercontent.com/Annie-XIE/summary-os/master/rdo_xenserver_helper.sh
    # sed -i 's/dom0_ip=169.254.0.1/dom0_ip=compute1/g' rdo_xenserver_helper.sh
5. Use the script to install the dom0 nova plugins::

    # source rdo_xenserver_helper.sh
    # install_dom0_plugins
* Answer yes to the RSA key prompt
* Enter the XenServer root password when prompted (twice)
* Ignore the errors related to the neutron plugins

6. Enable and start the nova services::

    # systemctl enable openstack-nova-compute.service
    # systemctl start openstack-nova-compute.service
7. Log on to the controller node as root.
8. Load the "admin" credential file::

    # source admin-openrc.sh
9. Check the nova service list::

    # nova service-list

      +----+------------------+---------------------------------------------+----------+---------+-------+----------------------------+-----------------+
      | Id | Binary           | Host                                        | Zone     | Status  | State | Updated_at                 | Disabled Reason |
      +----+------------------+---------------------------------------------+----------+---------+-------+----------------------------+-----------------+
      | 1  | nova-consoleauth | controller.openstack.lab.eco.rackspace.com  | internal | enabled | up    | 2016-02-08T16:53:19.000000 | -               |
      | 2  | nova-scheduler   | controller.openstack.lab.eco.rackspace.com  | internal | enabled | up    | 2016-02-08T16:53:19.000000 | -               |
      | 3  | nova-conductor   | controller.openstack.lab.eco.rackspace.com  | internal | enabled | up    | 2016-02-08T16:53:22.000000 | -               |
      | 4  | nova-cert        | controller.openstack.lab.eco.rackspace.com  | internal | enabled | up    | 2016-02-08T16:53:27.000000 | -               |
      | 5  | nova-compute     | compute1-vm.openstack.lab.eco.rackspace.com | nova     | enabled | up    | 2016-02-08T16:53:19.000000 | -               |
      +----+------------------+---------------------------------------------+----------+---------+-------+----------------------------+-----------------+

* The list should include ``compute1-vm`` running ``nova-compute``.

10. Check the nova endpoints list::

     # nova endpoints

       WARNING: nova has no endpoint in ! Available endpoints for this service:
       +-----------+------------------------------------------------------------+
       | nova      | Value
       +-----------+------------------------------------------------------------+
       | id        | 1c07bba299254336abd0cbe27c64be83                           |
       | interface | internal                                                   |
       | region    | RegionOne                                                  |
       | region_id | RegionOne                                                  |
       | url       | http://controller:8774/v2/76f8c8fd7b1e407d97c4604eb2a408b3 |
       +-----------+------------------------------------------------------------+
       +-----------+------------------------------------------------------------+
       | nova      | Value                                                      |
       +-----------+------------------------------------------------------------+
       | id        | 221f3238f2da46fb8fc6897e6c2c4de1                           |
       | interface | public                                                     |
       | region    | RegionOne                                                  |
       | region_id | RegionOne                                                  |
       | url       | http://controller:8774/v2/76f8c8fd7b1e407d97c4604eb2a408b3 |
       +-----------+------------------------------------------------------------+
       +-----------+------------------------------------------------------------+
       | nova      | Value                                                      |
       +-----------+------------------------------------------------------------+
       | id        | fdbd2fe1dda5460aaa486b5d142f99aa                           |
       | interface | admin                                                      |
       | region    | RegionOne                                                  |
       | region_id | RegionOne                                                  |
       | url       | http://controller:8774/v2/76f8c8fd7b1e407d97c4604eb2a408b3 |
       +-----------+------------------------------------------------------------+
       WARNING: keystone has no endpoint in ! Available endpoints for this service:
       +-----------+----------------------------------+
       | keystone  | Value                            |
       +-----------+----------------------------------+
       | id        | 33c74602793e454ea1d9ae9ab6ca5dcc |
       | interface | public                           |
       | region    | RegionOne                        |
       | region_id | RegionOne                        |
       | url       | http://controller:5000/v2.0      |
       +-----------+----------------------------------+
       +-----------+----------------------------------+
       | keystone  | Value                            |
       +-----------+----------------------------------+
       | id        | 688939b258ea4f1d956cb85dfc75e0c0 |
       | interface | internal                         |
       | region    | RegionOne                        |
       | region_id | RegionOne                        |
       | url       | http://controller:5000/v2.0      |
       +-----------+----------------------------------+
       +-----------+----------------------------------+
       | keystone  | Value                            |
       +-----------+----------------------------------+
       | id        | 7c7652f07b2f4a2c8bf805ff49b6a4eb |
       | interface | admin                            |
       | region    | RegionOne                        |
       | region_id | RegionOne                        |
       | url       | http://controller:35357/v2.0     |
       +-----------+----------------------------------+
       WARNING: glance has no endpoint in ! Available endpoints for this service:
       +-----------+----------------------------------+
       | glance    | Value                            |
       +-----------+----------------------------------+
       | id        | 0d49d35fc21d4faa8c72ff3578198513 |
       | interface | internal                         |
       | region    | RegionOne                        |
       | region_id | RegionOne                        |
       | url       | http://controller:9292           |
       +-----------+----------------------------------+
       +-----------+----------------------------------+
       | glance    | Value                            |
       +-----------+----------------------------------+
       | id        | 54f519365b8e4f7f81b750fdbf55be2f |
       | interface | public                           |
       | region    | RegionOne                        |
       | region_id | RegionOne                        |
       | url       | http://controller:9292           |
       +-----------+----------------------------------+
       +-----------+----------------------------------+
       | glance    | Value                            |
       +-----------+----------------------------------+
       | id        | d5e7d60a0eba46b9ac7b992214809fe0 |
       | interface | admin                            |
       | region    | RegionOne                        |
       | region_id | RegionOne                        |
       | url       | http://controller:9292           |
       +-----------+----------------------------------+
* The list should include endpoints for ``nova``, ``keystone``, and ``glance``. Ignore any warnings.

11. Check the nova image list::

     # nova image-list

       +--------------------------------------+----------------+--------+--------------------------------------+
       | ID                                   | Name           | Status | Server                               |
       | 1e710e0c-0fb6-4425-b196-4b66bfac495e | cirros-xen     | ACTIVE |                                      |
       +--------------------------------------+----------------+--------+--------------------------------------+
* The list should include the ``cirros-xen`` image previously uploaded.
