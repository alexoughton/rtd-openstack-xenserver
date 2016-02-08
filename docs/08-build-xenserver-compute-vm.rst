.. highlight:: none

8. Build XenServer Compute VM
=============================

This page is based on the following OpenStack Installation Guide pages:

http://docs.openstack.org/liberty/install-guide-rdo/environment-networking-compute.html

http://docs.openstack.org/liberty/install-guide-rdo/environment-ntp-other.html

http://docs.openstack.org/liberty/install-guide-rdo/environment-packages.html

**There are many additional steps here specific to XenServer.**

1. In XenCenter, create a new VM:

.. image:: assets/page08-create-new-vm.png
2. Select the CentOS 7 template:

.. image:: assets/page08-select-template.png
3. Name the VM "``compute``":

.. image:: assets/page08-name-vm.png
4. Choose the CentOS 7 ISO (which you should have previously uploaded to the ISO library):

.. image:: assets/page08-choose-iso.png
5. Place the VM on the only server available:

.. image:: assets/page08-choose-server.png
6. Give it one CPU and 2GB:

.. image:: assets/page08-cpu-ram.png
7. Change the disk to 20GB by clicking on properties:

.. image:: assets/page08-disk1.png

.. image:: assets/page08-disk2.png
8. Give the VM connections to your management and public networks:

.. image:: assets/page08-network.png
9. Complete the wizard, which will start the VM.
10. Go to the "compute" VM's console, which should be displaying the CentOS installer's boot screen:

.. image:: assets/page08-console.png
11. Highlight "Install CentOS 7", and press Enter:

.. image:: assets/page08-boot.png
12. If the console appears to "hang", with only a cursor showing (and no other activity), then quit XenCenter, relaunch it, and go back to the console. This should show the graphical installer is now running:

.. image:: assets/page08-installer-started.png
13. Set language and timezone.
14. Click on "Network & Hostname". Click on the "eth1" interface, and click on "configure".
15. Set the IPv4 address as appropriate:

.. image:: assets/page08-set-ipv4.png
16. Disable IPv6, and click on "save":

.. image:: assets/page08-disable-ipv6.png
17. Set an appropriate hostname, and then enable the "eth1" interface by setting the switch to "on":

.. image:: assets/page08-enable-interface.png
18. If using the NetInstall image, click on "Installation source".

* Set the source to network, and then define a known-good mirror. You can use http://mirror.rackspace.com/CentOS/7.2.1511/os/x86_64/

19. Click on "Installation Destination". Select "I will configure partitioning" and click on "Done":

.. image:: assets/page08-i-will-configure.png
20. Under "New mount points will use the following partition scheme", select "Standard Partition".
21. Click on the + button. Set the mount point to / and click "Add mount point":

.. image:: assets/page08-new-mount-point.png
22. Set "File System" to "ext4", and then click "Done".

.. image:: assets/page08-ext4.png
23. A yellow warning bar will appear. Click "Done" again, and then click on "Accept Changes".

.. image:: assets/page08-accept-changes.png
24. Click on "Software Selection". Select "Infrastructure Server", and click "Done".

.. image:: assets/page08-software-selection.png
25. Click "Begin Installation". Click on "Root Password" and set a good password.
26. Once installation is complete, click "Reboot".
27. SSH as root to the new VM.
28. In XenCenter, change the DVD drive to xs-tools.iso

.. image:: assets/page08-xs-tools-iso.png
29. Mount the tools ISO and install the tools::

     # mkdir /mnt/cdrom
     # mount /dev/cdrom /mnt/cdrom
     # cd /mnt/cdrom/Linux
     # rpm -Uvh xe-guest-utilities-6.5.0-1427.x86_64.rpm xe-guest-utilities-xenstore-6.5.0-1427.x86_64.rpm
     # cd ~
     # umount /mnt/cdrom
30. In XenCenter, eject the DVD drive

.. image:: assets/page08-eject.png
31. Stop and disable the firewalld service::

     # systemctl disable firewalld.service
     # systemctl stop firewalld.service
32. Disable SELINUX::

     # setenforce 0
     # vim /etc/sysconfig/selinux

       SELINUX=permissive
33. Update all packages on the VM::

     # yum update
34. Reboot the VM::

     # systemctl reboot
35. Wait for the VM to complete the reboot, and SSH back in as root.
36. Update the system hosts file with entries for all nodes::

     # vim /etc/hosts
       172.16.0.192 controller controller.openstack.lab.eco.rackspace.com
       172.16.0.203 compute1 compute1.openstack.lab.eco.rackspace.com
       172.16.0.204 compute1-vm compute1-vm.openstack.lab.eco.rackspace.com
       172.16.0.195 compute2 compute2.openstack.lab.eco.rackspace.com
       172.16.0.196 block1 block1.openstack.lab.eco.rackspace.com
       172.16.0.197 object1 object1.openstack.lab.eco.rackspace.com
       172.16.0.198 object2 object2.openstack.lab.eco.rackspace.com
37. Update the chrony configuration to use the controller as a time source::

     # vim /etc/chrony.conf

       server controller iburst
* Remove any other servers listed, leaving only "``controller``".

38. Restart the chrony service, and confirm that "``controller``" is listed as a source::

     # systemctl restart chronyd.service
     # chronyc sources
       210 Number of sources = 1
       MS Name/IP address         Stratum Poll Reach LastRx Last sample
       ===============================================================================
       ^* controller                    3   6    17     6  -3374ns[+2000ns] +/- 6895us
39. Enable the OpenStack-Liberty yum repository::

     # yum install centos-release-openstack-liberty
40. Install the OpenStack client and SELINUX support::

     # yum install python-openstackclient openstack-selinux
