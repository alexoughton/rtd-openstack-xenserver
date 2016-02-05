1. Build Controller Host
========================

1. Note: If using VMWare hypervisor to run the controller host as a VM, you must enable promiscuous mode on the vSwitches.
2. Boot with CentOS 7.2.1511 DVD.
3. Set timezone
4. Software selection: Infrastructure server
5. Keep automatic partitioning. Allow to install only on first disk.
6. Set IPv4 and hostname. Disable IPv6. Give connection eth1 name.
7. Begin installation
8. Set root password
9. Reboot and remove ISO
10. SSH in to server as root
11. Stop and disable the firewalld service::

     # systemctl disable firewalld.service
     # systemctl stop firewalld.servvice
13. # setenforce 0
14. # vim /etc/sysconfig/selinux
SELINUX=permissive
15. # yum update
16. # yum install open-vm-tools

17. # vim /etc/udev/rules.d/90-persistent-net.rules
 Change these MAC addresses to match your own::

  SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*",ATTR{address}=="00:0c:29:d9:36:46",ATTR{dev_id}=="0x0", ATTR{type}=="1",KERNEL=="eno*", NAME="eth0"
  SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*",ATTR{address}=="00:0c:29:d9:36:50",ATTR{dev_id}=="0x0", ATTR{type}=="1",KERNEL=="eno*", NAME="eth1"
18. # cd /etc/sysconfig/network-scripts
19. # mv ifcfg-eno16777984 ifcfg-eth0
20. # mv ifcfg-eno33557248 ifcfg-eth1
21. # vim ifcfg-eth0
Change all references of eno16777984 to eth0
22. # vim ifcfg-eth1
Change all references of eno33557248 to eth1
23. # systemctl reboot

24. SSH back in as root after the reboot
25. Check that ifconfig now shows eth0 and eth1.
26. Update the system hosts file with enties for all nodes::

    # vim /etc/hosts

    172.16.0.192 controller controller.openstack.lab.eco.rackspace.com
    172.16.0.203 compute1 compute1.openstack.lab.eco.rackspace.com
    172.16.0.204 compute1-vm compute1-vm.openstack.lab.eco.rackspace.com
    172.16.0.195 compute2 compute2.openstack.lab.eco.rackspace.com
    172.16.0.196 block1 block1.openstack.lab.eco.rackspace.com
    172.16.0.197 object1 object1.openstack.lab.eco.rackspace.com
    172.16.0.198 object2 object2.openstack.lab.eco.rackspace.com
bort

27. # vim /etc/chrony.conf
Allow 172.16.0.0/24

28. # systemctl restart chronyd.service
29. # yum install centos-release-openstack-liberty
30. # yum install python-openstackclient openstack-selinux
