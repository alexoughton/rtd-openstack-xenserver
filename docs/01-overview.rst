.. highlight:: none

1. Overview
===========

The OpenStack foundation has an excellent setup guide for their October 2015 release, "Liberty",
which can be found at http://docs.openstack.org/liberty/install-guide-rdo/. However, this guide
only deals with the use of the "KVM" hypervisor, and does not cover the use of "XenServer" hypervisor.

There are many circumstances in which it may be desirable to build an OpenStack Liberty XenServer
environment, however in my efforts to do so, I have found the available online documentation
regarding using XenServer with OpenStack to be inadequate, outdated or just plain incorrect.
Specifically, during this project I experienced issues with:

* XenServer networking configuration
* Nova and Neutron configurations for XenServer networking
* iSCSI authentication issues with Cinder volumes
* Cinder volume mapping errors with XenServer instances
* Cinder quota errors
* ISO image support for XenServer
* Horizon bug affecting XenServer images
* Image metadata for dual hypervisor-type environments
* Neutron requirements for dual-hypervisor-type environments
* Neutron bug affecting the use of OpenvSwitch (Required for XenServer)
* VNC console connectivity

This guide is heavily based on the OpenStack foundation's guide, however it does not go
into the same level of detail. Their guide should be considered the superior one, and the
"master" guide. However, this guide can be referenced in order to highlight the differences
when using XenServer.

Some elements of this guide are also based on the following blog post:
https://www.citrix.com/blogs/2015/11/30/integrating-xenserver-rdo-and-neutron/

On each page, I have highlighted in **bold** any steps which differ from the original guide.
These are typically XenServer-specific changes.

This guide is for a simple setup with "flat" networking. There are no provisions for private
"virtual" networks, or any firewall functionality. The guide also does not yet cover "swift"
object storage, although this shouldn't differ from the OpenStack foundation's guide. A future
version of the guide may add these functions.

Later pages in this guide deal with adding a KVM hypervisor to the environment. These pages include
changes which I found to be necessary in order to support a dual hypervisor-type environment (i.e
the use of XenServer and KVM in the same OpenStack).

Two networks are required, a "public" network (which instances will be connected to for their
day-to-day traffic), and a "management" network, which our OpenStack servers will use for their
connectivity. Any servers with connections to both will have eth0 connected to the "public" network,
and eth1 connected to the "management" network.

Any IP addresses in the guide should, of course, be replaced with your own. You will also need to
pre-generate the following variables which will be referred to throughout the guide:

============  =======
 Variable     Meaning
============  =======
False  False  False
True   False  True
False  True   True
True   True   True
============  =======

=============================  =========================================================================================================================
 Variable                      Meaning
=============================  =========================================================================================================================
``*MYSQL_ROOT*``               Root password for MySQL
``*KEYSTONE_DBPASS*``          Password for the ``keystone`` MySQL database.
``*ADMIN_TOKEN*``              A temporary token for initial connection to keystone. Can be created by running ``openssl rand -hex 10``.
``*RABBIT_PASS*``              Password for the ``openstack`` rabbitmq user.
``*GLANCE_DBPASS*``            Password for the ``glance`` MySQL database.
``*GLANCE_PASS*``              Password for the ``glance`` identity user.
``*NOVA_DBPASS*``              Password for the ``nova`` MySQL database.
``*NOVA_PASS*``                Password for the ``nova`` identity user.
``*NEUTRON_DBPASS*``           Password for the ``neutron`` MySQL database.
``*NEUTRON_PASS*``             Password for the ``neutron`` identity user.
``*NEUTRON_METADATA_SECRET*``  Random secret string for the metadata service.
``*CINDER_DBPASS*``            Password for the ``cinder`` MySQL database.
``*CINDER_PASS*``              Password for the ``cinder`` identity user.
``*XENSERVER_ROOT*``           Root password for XenServer. Do not use a password you're not comfortable placing in plaintext in the nova configuration.
``*XENSERVER_IP*``             IP address of XenServer.
``*CONTROLLER_ADDRESS*``       A DNS address for the controller server which can be reached from your workstation.
``*ADMIN_PASS*``               Password for the ``admin`` identity user.
``*DEMO_PASS*``                Password for the ``demo`` identity user.
=============================  =========================================================================================================================
