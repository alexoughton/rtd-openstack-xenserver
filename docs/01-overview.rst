.. highlight:: none

1. Overview
===========

The OpenStack foundation has an excellent setup guide for their October 2015 release, "Liberty",
which can be found at http://docs.openstack.org/liberty/install-guide-rdo/. However, this guide
only deals with the use of the "KVM" hypervisor, and does not cover the use of "XenServer" hypervisor.

There are many circumstances in which it may be desirable to build an OpenStack Liberty XenServer
environment. However, in my efforts to do so, I have found the available online documentation
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

This guide is heavily based on the OpenStack foundation's guide. It does not go
into the same level of detail, but does highlight the differences when using
XenServer instead of KVM. Their guide should be considered the superior one, and the
"master" guide, and I recommend reading their guide if you have no familiarity with
OpenStack at all.

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

Finally, there are pages regarding the creation of CentOS 7 images for both hypervisors.
These pages highlight some differences in the image-creation process for both hypervisors,
including the package and partitioning requirements to support automatic disk resizing
and injection of SSH keys for the root user.

Two networks are required, a "public" network (which instances will be connected to for their
day-to-day traffic), and a "management" network, which our OpenStack servers will use for their
connectivity. Any servers with connections to both will have eth0 connected to the "public" network,
and eth1 connected to the "management" network.

Any IP addresses in the guide should, of course, be replaced with your own. You will also need to
pre-generate the following variables which will be referred to throughout the guide:

=============================  =====================================================
 Variable                      Meaning
=============================  =====================================================
``*MYSQL_ROOT*``               Root password for MySQL.
``*KEYSTONE_DBPASS*``          Password for the ``keystone`` MySQL database.
``*ADMIN_TOKEN*``              A temporary token for initial connection to keystone.
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
``*XENSERVER_ROOT*``           Root password for XenServer.
``*XENSERVER_IP*``             IP address of XenServer.
``*CONTROLLER_ADDRESS*``       A DNS address for the controller server.
``*ADMIN_PASS*``               Password for the ``admin`` identity user.
``*DEMO_PASS*``                Password for the ``demo`` identity user.
``*XAPI_BRIDGE*``              The name of the ovs bridge to be used by instances.
=============================  =====================================================

* The ``*ADMIN_TOKEN*`` can be created by running::

   # openssl rand -hex 10
* For ``*XENSERVER_ROOT*``, do not use a password you're not comfortable placing in plaintext in the nova configuration.

* For ``*CONTROLLER_ADDRESS*``, ensure that this is an address which you can reach from your workstation.

* For ``*XAPI_BRIDGE*``, this won't be determined until later in the builld process. You should write it down for later use once it is defined.

Also note that any instance of "``*SERVER_IP*``" refers to the server you are currently working on. Any instance of
"``*VM_IP*``" refers to the IP of the XenServer compute VM.

One final note: I do disable SELINUX in this guide, for simplicity. This is a personal choice,
but I know that some people do choose to run SELINUX on their systems. The guide does include
the installation of SELINUX support for openstack, so you should be able to set this back to "``ENFORCING``",
even after performing the installation with this set to "``PERMISSIVE``". I have not tested this.

Changelog
---------

Mar 9 2016:
 * Add note regarding case-sensitive udev rules file.

Mar 4 2016:
 * Add fix to prevent installation of kernels from Xen repository on Storage node.

Feb 19 2016:
 * Add fix to Horizon config for Identity v3.
 * Fix changelog order.

Feb 17 2016:
 * Add steps to enable auto power-on of the "compute" VM on the XenServer host.
 * Add required steps to enable migration and live migration of instances between XenServer hosts.

Feb 12 2016:
 * Create changelog.
 * Various clarifications.
 * Extended identity's token expiration time.
 * Correct syntax for neutron ovs configuration on controller.
 * Correct syntax when populating neutron database.
 * Add note regarding large storage requirements for cinder image-to-volume conversion.

About the Author
----------------

My name is Alex Oughton, and I work with OpenStack clouds, as well as dedicated hosting solutions.
My work doesn't involve the actual deployment of OpenStack, and so this guide was developed during
a self-learning exercise. If you have any feedback regarding this guide, including any suggestions
or fixes, please do contact me on Twitter: http://twitter.com/alexoughton.

You can also directly contribute to this guide through its github: https://github.com/alexoughton/rtd-openstack-xenserver.
