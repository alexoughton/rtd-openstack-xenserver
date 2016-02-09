.. highlight:: none

1. Overview
===========

The OpenStack foundation has an excellent setup guide for their October 2015 release, "Liberty", which can be found at http://docs.openstack.org/liberty/install-guide-rdo/. However, this guide only deals with the use of the "KVM" hypervisor, and does not cover the use of "XenServer" hypervisor.

There are many circumstances in which it may be desirable to build an OpenStack Liberty XenServer environment, however in my efforts to do so, I have found the available online documentation regarding using XenServer with OpenStack to be inadequate, outdated or just plain incorrect. Specifically, during this project I experienced issues with:

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
