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

## 5- Validation
### Validate through dashboard
* Compute services are enabled and up
![image](https://user-images.githubusercontent.com/47721226/218250205-2554cb28-a0c0-4144-8505-1e59771979d2.png)

* Network services are enabled and up
![image](https://user-images.githubusercontent.com/47721226/218250329-0f2bd6ef-910b-4532-b548-a6f58073cc17.png)

* Storage services are enabled and up
![image](https://user-images.githubusercontent.com/47721226/218250381-ee8935ba-f1b7-444d-988e-b7923bcd5132.png)

* each service installed has three end-points
![image](https://user-images.githubusercontent.com/47721226/218250069-4d0a20a9-1299-48a2-830e-52641aedb16c.png)

### Validate VM creation
* Create a private network `neutron net-create Internal`
* Create a subnet attached to the private network `openstack subnet create --network Internal --subnet-range 192.168.11.0/24 --dhcp Internal_subnet`
* Create an external network, so that instances can be reachable through internet `openstack subnet create External_subnet --no-dhcp --allocation-pool start=192.168.206.140,end=192.168.206.150 --gateway 192.168.206.2 --network External --subnet-range 192.168.206.0/24`
* Create router to connect the external network and the internal network to each other `neutron router-create router`
* Set its getway using the external network `neutron router-gateway-set router External`
* Connect the private network to the router `neutron router-interface-add router Internal_subnet`
![image](https://user-images.githubusercontent.com/47721226/218251194-0316669e-9f2a-4d16-ad41-767a4cd89eb1.png)
* Install image file and Create image using glance component `curl -L http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img | glance image-create --name='cirros image' --visibility=public --container-format=bare --disk-format=qcow2`
* Create an instance `openstack server create --image 'cirros image' --flavor m1.tiny  --network Internal server1`
* Create a floating ip to be able to reach that instance from the external network `openstack floating ip create External`
* Assign the floating ip to the VM port `openstack floating ip set --port 64cb1b63-be9a-481b-9a81-c1aca04b3386 192.168.206.149`
* Add two rules in the default security group to enable ssh port and ICMP
![image](https://user-images.githubusercontent.com/47721226/218251509-bb0507e8-9eed-4c6b-b6aa-1ba72328a485.png)
* SSH to the instance using the floating ip
![image](https://user-images.githubusercontent.com/47721226/218251648-26443200-8ab2-4d4c-8fa1-14df0ab69cf5.png)
* Ping to the instance using the floating ip
![image](https://user-images.githubusercontent.com/47721226/218251995-83717d71-1037-44d1-8f3c-89b364f626eb.png)

