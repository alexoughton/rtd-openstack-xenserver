.. highlight:: none

12. Install Dashboard (horizon) on controller
=============================================

This page is based on the following OpenStack Installation Guide page:

http://docs.openstack.org/liberty/install-guide-rdo/horizon-install.html

**Step 3 has specific changes for the use of XenServer.**

1. Install horizon packages::

    # yum install openstack-dashboard
2. Configure horizon::

    # vim /etc/openstack-dasboard/local_settings

      OPENSTACK_CONTROLLER = "controller"
      ALLOWED_HOSTS = ['*', ]
      CACHES = {
          'default': {
               'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
               'LOCATION': '127.0.0.1:11211',
          }
      }
      OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
      OPENSTACK_NEUTRON_NETWORK = {
          'enable_router': False,
          'enable_quotas': False,
          'enable_distributed_router': False,
          'enable_ha_router': False,
          'enable_lb': False,
          'enable_firewall': False,
          'enable_vpn': False,
          'enable_fip_topology_check': False,
      }
      TIME_ZONE = "*TIME_ZONE*"
* Note 1: There are many options already present in the file. These should be left as-is.
* Note 2: For the ``openstack_neutron_network`` block, modify the settings listed above, rather than replacing the entire block.

3. There is a bug in Horizon which is breaking image metadata when editing XenServer images. This has been reported in https://bugs.launchpad.net/horizon/+bug/1539722. Until the bug is fixed, here is a quick and dirty patch to avoid the problem:

   a. Bort

   b. Bort 2
