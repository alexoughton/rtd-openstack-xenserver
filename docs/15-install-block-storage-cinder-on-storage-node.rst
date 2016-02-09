.. highlight:: none

15. Install Block Storage (cinder) on storage node
==================================================

This page is based on the following OpenStack Installation Guide page:

http://docs.openstack.org/liberty/install-guide-rdo/cinder-storage-install.html

**Steps 3, 4, 5, 8 and 9 have specific changes for the use of XenServer.**

1. Create the LVM volume group on the second disk::

    # pvcreate /dev/sdb
    # vgcreate cinder-volumes /dev/sdb
2. Update the LVM configuration to prevent scanning of cinder volumes' contents::

    # vim /etc/lvm/lvm.conf

      devices {
         ...
         filter = [ "a/sda/", "a/sdb/", "r/.*/"]
* Note: Do not replace the entire "``devices``" section, only the "``filter``" line.

3. Enable the ``centos-release-xen`` and ``epel-release`` repositories::

    # yum install centos-release-xen epel-release
4. Install special packages needed from outside of the openstack-liberty repositories::

    # yum install scsi-target-utils xen-runtime
5. Remove the ``epel-release`` repository again::

    # yum remove epel-release
6. Install the cinder packages::

    # yum install openstack-cinder python-oslo-policy
7. Configure cinder. Replace ``*CINDER_DBPASS*``, ``*SERVER_IP*``, ``*RABBIT_PASS*`` and ``*CINDER_PASS*`` with your own::

    # vim /etc/cinder/cinder.conf

      [database]
      connection = mysql://cinder:*CINDER_DBPASS*@controller/cinder

      [DEFAULT]
      rpc_backend = rabbit
      auth_strategy = keystone
      my_ip = *SERVER_IP*
      enabled_backends = lvm
      glance_host = controller

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
      username = cinder
      password = *CINDER_PASS*

      [lvm]
      volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
      volume_group = cinder-volumes
      iscsi_protocol = iscsi
      iscsi_helper = tgtadm

      [oslo_concurrency]
      lock_path = /var/lib/cinder/tmp

8. Update the tgtd.conf configuration. There are other lines in this file. Don't change those, just add this one::

    # vim /etc/tgt/tgtd.conf

      include /var/lib/cinder/volumes/*
9. Enable and start the tgtd and cinder services::

    # systemctl enable tgtd.service openstack-cinder-volume.service
    # systemctl start tgtd.service openstack-cinder-volume.service
