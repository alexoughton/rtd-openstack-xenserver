.. highlight:: none

14. Install Block Storage (cinder) on controller
================================================

This page is based on the following OpenStack Installation Guide page:

http://docs.openstack.org/liberty/install-guide-rdo/cinder-controller-install.html

1. Open the MySQL client and create the "cinder" database. Replace ``*CINDER_DBPASS*`` with your own::

    # mysql
      > create database cinder;
      > grant all privileges on cinder.* to 'cinder'@'localhost' identified by '*CINDER_DBPASS*';
      > grant all privileges on cinder.* to 'cinder'@'%' identified by '*CINDER_DBPASS*';
      > quit
2. Create the "cinder" user, role, services and endpoints. Provide ``*CINDER_PASS*`` when prompted::

    # source admin-openrc.sh
    # openstack user create --domain default --password-prompt cinder
    # openstack role add --project service --user cinder admin
    # openstack service create --name cinder --description "OpenStack Block Storage" volume
    # openstack service create --name cinderv2 --description "OpenStack Block Storage" volumev2
    # openstack endpoint create --region RegionOne volume public http://controller:8776/v1/%\(tenant_id\)s
    # openstack endpoint create --region RegionOne volume internal http://controller:8776/v1/%\(tenant_id\)s
    # openstack endpoint create --region RegionOne volume admin http://controller:8776/v1/%\(tenant_id\)s
    # openstack endpoint create --region RegionOne volumev2 public http://controller:8776/v2/%\(tenant_id\)s
    # openstack endpoint create --region RegionOne volumev2 internal http://controller:8776/v2/%\(tenant_id\)s
    # openstack endpoint create --region RegionOne volumev2 admin http://controller:8776/v2/%\(tenant_id\)s
3. Install the cinder packages::

    # yum install openstack-cinder python-cinderclient
4. Configure cinder. Replace ``*SERVER_IP*``, ``*CINDER_DBPASS*``, ``*CINDER_PASS*`` and ``*RABBIT_PASS*`` with your own::

    # vim /etc/cinder/cinder.conf

      [database]
      connection = mysql://cinder:*CINDER_DBPASS*@controller/cinder

      [DEFAULT]
      rpc_backend = rabbit
      auth_strategy = keystone
      my_ip = *SERVER_IP*
      nova_catalog_info = compute:nova:publicURL
      nova_catalog_admin_info = compute:nova:adminURL

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

      [oslo_concurrency]
      lock_path = /var/lib/cinder/tmp

5. Populate the cinder database::

    # su -s /bin/sh -c "cinder-manage db sync" cinder
6. Reconfigure nova for cinder::

    # vim /etc/nova/nova.conf

      [cinder]
      os_region_name = RegionOne
7. Restart the nova service::

    # systemctl restart openstack-nova-api.service
8. Enable and start the cinder services::

    # systemctl enable openstack-cinder-api.service openstack-cinder-scheduler.service
    # systemctl start openstack-cinder-api.service openstack-cinder-scheduler.service
