.. highlight:: none

5. Install Images (glance) on controller
========================================

This page is based on the following OpenStack Installation Guide pages:

http://docs.openstack.org/liberty/install-guide-rdo/glance-install.html

http://docs.openstack.org/liberty/install-guide-rdo/glance-verify.html

Step 9 has specific changes for the use of XenServer.

1. Open the MySQL client and create the keystone database. Replace \*GLANCE_DBPASS\* with your own::

	  # mysql
      > create database glance;
      > grant all privileges on glance.* to 'glance'@'localhost' identified by '*GLANCE_DBPASS*';
      > grant all privileges on glance.* to 'glance'@'%' identified by '*GLANCE_DBPASS*';
      > quit
2. Create the "glance" user, role, service and endpoints. Provide \*GLANCE_PASS\* when prompted::

    # source admin-openrc.sh
    # openstack user create --domain default --password-prompt glance
    # openstack role add --project service --user glance admin
    # openstack service create --name glance --description "OpenStack Image service" image
    # openstack endpoint create --region RegionOne image public http://controller:9292
    # openstack endpoint create --region RegionOne image internal http://controller:9292
    # openstack endpoint create --region RegionOne image admin http://controller:9292
3. Install glance packages::

	  # yum install openstack-glance python-glance python-glanceclient
4. Configure glance-api. Replace \*GLANCE_DBPASS\* and \*GLANCE_PASS\* with your own::

	  # vim /etc/glance/glance-api.conf

	    [database]
	    connection = mysql://glance:*GLANCE_DBPASS*@controller/glance

	    [keystone_authtoken]
	    auth_uri = http://controller:5000
	    auth_url = http://controller:35357
	    auth_plugin = password
	    project_domain_id = default
	    user_domain_id = default
	    project_name = service
	    username = glance
	    password =  *GLANCE_PASS*

    	[paste_deploy]
	    flavor = keystone

	    [glance_store]
	    default_store = file
	    filesystem_store_datadir = /var/lib/glance/images/

	    [DEFAULT]
	    notification_driver = noop

5. Configure glance-registry. Replace \*GLANCE_DBPASS\* and \*GLANCE_PASS\* with your own::

	  # vim /etc/glance/glance-registry.conf

	    [database]
	    connection = mysql://glance:*GLANCE_DBPASS*@controller/glance

	    [keystone_authtoken]
	    auth_uri = http://controller:5000
	    auth_url = http://controller:35357
	    auth_plugin = password
	    project_domain_id = default
	    user_domain_id = default
	    project_name = service
	    username = glance
	    password = *GLANCE_PASS*

	    [paste_deploy]
	    flavor=keystone

	    [DEFAULT]
	    Notification_driver = noop

6. Populate the glance database::

	  # su -s /bin/sh -c "glance-manage db_sync" glance

7. Enable and start the glance service::

    # systemctl enable openstack-glance-api.service openstack-glance-registry.service
    # systemctl start openstack-glance-api.service openstack-glance-registry.service
8. Add glance API version settings to the client authentication files::

	  # echo "export OS_IMAGE_API_VERSION=2" | tee -a admin-openrc.sh demo-openrc.sh
9. **Upload a sample image to the glance service**::

    # source admin-openrc.sh
    # wget http://ca.downloads.xensource.com/OpenStack/cirros-0.3.4-x86_64-disk.vhd.tgz
    # glance image-create --name "cirros-xen" --container-format ovf --disk-format vhd --property vm_mode=xen --visibility public --file cirros-0.3.4-x86_64-disk.vhd.tgz
10. Confirm that the image has been uploaded::

     # glance image-list

        +--------------------------------------+----------------+
        | ID                                   | Name           |
        +--------------------------------------+----------------+
        | 1e710e0c-0fb6-4425-b196-4b66bfac495e | cirros-xen     |
        +--------------------------------------+----------------+
