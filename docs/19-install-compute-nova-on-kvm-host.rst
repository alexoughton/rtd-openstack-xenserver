.. highlight:: none

19. Install Compute (nova) on KVM Host
======================================

This page is based on the following OpenStack Installation Guide pages:

http://docs.openstack.org/liberty/install-guide-rdo/nova-compute-install.html

http://docs.openstack.org/liberty/install-guide-rdo/cinder-storage-install.html

http://docs.openstack.org/liberty/install-guide-rdo/nova-verify.html

1. Install nova packages::

    # yum install openstack-nova-compute sysfsutils
2. Format and mount the second array for instance storage::

    # parted -s -- /dev/sdb mklabel gpt
    # parted -s -a optimal -- /dev/sdb mkpart primary 2048s -1
    # parted -s -- /dev/sdb align-check optimal 1
    # parted /dev/sdb set 1 lvm on
    # parted /dev/sdb unit s print
    # mkfs.xfs /dev/sdb1
    # mount /dev/sdb1 /var/lib/nova/instances
    # tail -1 /etc/mtab >> /etc/fstab
    # chown nova:nova /var/lib/nova/instances
3. Update the LVM configuration to prevent scanning of instances' contents::

    # vim /etc/lvm/lvm.conf

      devices {
         ...
         filter = [ "a/sda/", "a/sdb/", "r/.*/"]
* Note: Do not replace the entire "``devices``" section, only the "``filter``" line.

4. Configure nova. Replace ``*SERVER_IP*``, ``*RABBIT_PASS*``, ``*NOVA_PASS*`` and ``*CONTROLLER_ADDRESS*`` with your own::

    # vim /etc/nova/nova.conf

      [DEFAULT]
      rpc_backend = rabbit
      auth_strategy = keystone
      my_ip = *SERVER_IP*
      network_api_class = nova.network.neutronv2.api.API
      security_group_api = neutron
      linuxnet_interface_driver = nova.network.linux_net.NeutronLinuxBridgeInterfaceDriver
      firewall_driver = nova.virt.firewall.NoopFirewallDriver

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
      vncserver_proxyclient_address = $my_ip
      novncproxy_base_url = http://*CONTROLLER_ADDRESS*:6080/vnc_auto.html

      [glance]
      host = controller

      [oslo_concurrency]
      lock_path = /var/lib/nova/tmp

      [libvirt]
      virt_type = kvm

5. Enable and start the nova and libvirt services::

    # systemctl enable libvirtd.service openstack-nova-compute.service
    # systemctl start libvirtd.service openstack-nova-compute.service

6. Log on to the control node as root.
7. Load the "admin" credential file::

    # source admin-openrc.sh
8. Check the nova service list::

     # nova service-list

       +----+------------------+---------------------------------------------+----------+---------+-------+----------------------------+-----------------+
       | Id | Binary           | Host                                        | Zone     | Status  | State | Updated_at                 | Disabled Reason |
       +----+------------------+---------------------------------------------+----------+---------+-------+----------------------------+-----------------+
       | 1  | nova-consoleauth | controller.openstack.lab.eco.rackspace.com  | internal | enabled | up    | 2016-02-09T17:19:38.000000 | -               |
       | 2  | nova-scheduler   | controller.openstack.lab.eco.rackspace.com  | internal | enabled | up    | 2016-02-09T17:19:41.000000 | -               |
       | 3  | nova-conductor   | controller.openstack.lab.eco.rackspace.com  | internal | enabled | up    | 2016-02-09T17:19:41.000000 | -               |
       | 4  | nova-cert        | controller.openstack.lab.eco.rackspace.com  | internal | enabled | up    | 2016-02-09T17:19:38.000000 | -               |
       | 5  | nova-compute     | compute1-vm.openstack.lab.eco.rackspace.com | nova     | enabled | up    | 2016-02-09T17:19:39.000000 | -               |
       | 6  | nova-compute     | compute2.openstack.lab.eco.rackspace.com    | nova     | enabled | up    | 2016-02-09T17:19:36.000000 | -               |
       +----+------------------+---------------------------------------------+----------+---------+-------+----------------------------+-----------------+
* The list should include ``compute1-vm`` and ``compute2`` running ``nova-compute``.
