.. highlight:: none

9. Install Compute (nova) on XenServer compute VM
=================================================

This page is based on the following OpenStack Installation Guide pages:

http://docs.openstack.org/liberty/install-guide-rdo/nova-compute-install.html

It is also based on some steps from the following guide:

https://www.citrix.com/blogs/2015/11/30/integrating-xenserver-rdo-and-neutron/

**All steps have modifications for XenServer.**

1. Download and install pip, and xenapi::

    # wget https://bootstrap.pypa.io/get-pip.py
    # python get-pip.py
    # pip install xenapi
2. Install nova packages::

    # yum install openstack-nova-compute sysfsutils
3. Configure nova. Replace ``*XENSERVER_ROOT*``, ``*CONTROLLER_ADDRESS*``, ``*XAPI_BRIDGE*``, ``*VM_IP*``, ``*NOVA_PASS*``, ``*XENSERVER_IP*`` and ``*RABIT_PASS*`` with your own::

    # vim /etc/nova/nova.conf

      [DEFAULT]
      rpc_backend = rabbit
      auth_strategy = keystone
      my_ip = *VM_IP*
      network_api_class = nova.network.neutronv2.api.API
      security_group_api = neutron
      linuxnet_interface_driver = nova.network.linux_net.NeutronLinuxBridgeInterfaceDriver
      firewall_driver = nova.virt.firewall.NoopFirewallDriver
      compute_driver = xenapi.XenAPIDriver

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
      username = nova
      password = *NOVA_PASS*

      [vnc]
      enabled = True
      vncserver_listen = 0.0.0.0
      vncserver_proxyclient_address = *IP ADDRESS OF XENSERVER*
      novncproxy_base_url = http://*CONTROLLER_ADDRESS*:6080/vnc_auto.html

      [glance]
      host = controller

      [oslo_concurrency]
      lock_path = /var/lib/nova/tmp

      [xenserver]
      connection_url=http://compute1
      connection_username=root
      connection_password=*XENSERVER_ROOT*
      vif_driver=nova.virt.xenapi.vif.XenAPIOpenVswitchDriver
      ovs_int_bridge=*XAPI_BRIDGE*
      ovs_integration_bridge=*XAPI_BRIDGE*
4. Download and modify a helper script for installing the dom0 plugins::

    # wget --no-check-certificate https://raw.githubusercontent.com/Annie-XIE/summary-os/master/rdo_xenserver_helper.sh
    # sed -i 's/dom0_ip=169.254.0.1/dom0_ip=compute1/g' rdo_xenserver_helper.sh
5. Install the script to install the dom0 plugins::

    # source rdo_xenserver_helper.sh
    # install_dom0_plugins
* Answer yes to the RSA key prompt
* Enter the XenServer root password when prompted (twice)
* Ignore the errors related to the neutron plugins

6. Enable and start the nova services::

    # systemctl enable openstack-nova-compute.service
    # systemctl start openstack-nova-compute.service
