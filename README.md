# openstack-installation-via-packstack

## 1-Hosts preparation 
we will need two VMs
* controller and network node: 6GB RAM, 4 cpu, 2 nics and 20G disk
* compute node: 4GB RAM, 4 cpu, 2 nics and 20G disk

## 2- Hosts configuration
this configuration will be done on both nodes
* stop firewall `systemctl disable firewalld` , `systemctl stop firewalld`
* stop network manager `systemctl disable NetworkManager` , `systemctl stop NetworkManager`
* install network-scripts `dnf install -y network-scripts`
* enable network service `dnf install -y network-scripts`
* disable selinux in `/etc/selinx/config` set `SELINUX=disabled`
* in `/etc/hosts` add `192.168.206.129 controller.mariamgad.local controller` `192.168.206.131 compute.mariamgad.local compute`

* Install the release of openstack package `dnf config-manager --enable powertools` , `dnf install -y centos-release-openstack-yoga`
*  Update the system `dnf -y update`

this step will be done only on the controller node
* Install packstack installer `dnf install -y openstack-packstack`

## 3- Set passwordless authentication from controller node to compute node 
on controller node run
* `ssh-keygen`
* `ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.206.131`

## 4- Generate and customize answer file 
* `packstack --gen-answer-file=/root/answer.txt`
* `vim /root/answer.txt`

`CONFIG_CONTROLLER_HOST=192.168.206.129`
`CONFIG_COMPUTE_HOSTS=192.168.206.131`
`CONFIG_NETWORK_HOSTS=192.168.206.129`
`CONFIG_PROVISION_DEMO=n`
`CONFIG_CEILOMETER_INSTALL=n`
`CONFIG_NEUTRON_L2_AGENT=openvswitch`
`CONFIG_NEUTRON_ML2_MECHANISM_DRIVERS=openvswitch`
`CONFIG_NEUTRON_ML2_TENANT_NETWORK_TYPES=vxlan`
`CONFIG_NEUTRON_ML2_TYPE_DRIVERS=vxlan,flat`
`CONFIG_NEUTRON_OVS_BRIDGE_MAPPINGS=extnet:br-ex`
`CONFIG_NEUTRON_OVS_BRIDGE_IFACES=br-ex:eth0`
