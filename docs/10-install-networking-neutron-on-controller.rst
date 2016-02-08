.. highlight:: none

10. Install Networking (neutron) on controller
==============================================

This page is based on the following OpenStack Installation Guide page:

http://docs.openstack.org/liberty/install-guide-rdo/neutron-controller-install.html

**Steps 13, 17, and 24-27 have specific changes for the use of XenServer.**

1. Open the MySQL client and create the "glance" database. Replace ``*GLANCE_DBPASS*`` with your own::

	  # mysql
      > create database
