Installing OpenStack and Nuage using Ubuntu MAAS/Juju
========================================================
Ubuntu Metal as a Service (MAAS) is a system to setup physical hardware for installing complex software products such as OpenStack. Juju is the service orchestration tool that helps you easily build cloud environments. Juju can be used to deploy applications directly on a MAAS cluster.

Please see the instructions below for installing OpenStack and Nuage on bare metal by using MAAS and Juju.

Create MAAS Cluster
--------------------
First you need to setup a MAAS server by following the instructions at http://maas.ubuntu.com/docs1.5/. This involves installing necessary packages, setting up the cluster, assigning networks etc. MAAS can use IPMI to do the power control operations and this requires setting up corresponding "lights out management" systems. Please refer server documentation for instructions.

Provisioning machines
-----------------------
Once the MAAS server is built, you can provision individual machines from the MAAS GUI. MAAS supports auto discovery of machines which means you can simply do PXE boot a machine in the network and it will appear at the MAAS GUI. There are various states for each machine in the cluster. They are "declared", "ready" and "allocated to admin". When MAAS discovers a machine it usually is in the "declared" state. Then you need to commission and start the node to get in the "ready" state. Before we do the OpenStack install, we want all machines to be in the "ready" state.

Install and Configure Juju
---------------------------
Instal Juju by following the ubuntu documentation at https://juju.ubuntu.com/docs/. After installing Juju, you also need to setup the Juju enviornment by following the same documentation.

Installing OpenStack using Juju Charms
--------------------------------------
Juju charms are a collection of software components that allow you to deploy and configure services on a cloud environment. Charms can be installed from external or local repositories. The OpenStack software can be installed using Juju charms in a variety of ways. Following shows an example deployment scenario:

### Create a config file

Create a configuration file with the following contents:

```
mysql:
 openstack-origin: cloud:precise-havana
rabbitmq-server:
 openstack-origin: cloud:precise-havana
glance:
 openstack-origin: cloud:precise-havana
openstack-dashboard:
 openstack-origin: cloud:precise-havana
keystone:
 admin-password: 'openstack'
 admin-token: 'ubuntutesting'
 openstack-origin: cloud:precise-havana
nova-cloud-controller:
 openstack-origin: cloud:precise-havana
 network-manager: Neutron
nova-compute:
 openstack-origin: cloud:precise-havana
 migration-auth-type: ssh
quantum-gateway:
 openstack-origin: cloud:precise-havana
 ext-port: eth1
cinder:
 openstack-origin: cloud:precise-havana
 block-device: "sdb"
 overwrite: "true"
cinder-volume:
 openstack-origin: cloud:precise-havana
 block-device: "sdb"
 overwrite: "true"
```

### Install OpenStack Charms and establish relationships
```
juju deploy cs:precise/mysql
juju deploy cs:precise/rabbitmq-server
juju deploy --config=<config file> cs:precise/keystone
juju deploy --config=<config file> cs:precise/nova-compute
juju deploy --config=<config file> cs:precise/nova-cloud-controller
juju deploy --config=<config file> cs:precise/glance
juju deploy --config=<config file> cs:precise/cinder
juju deploy --config=<config file> cs:precise/quantum-gateway
juju deploy --config=<config file> cs:precise/openstack-dashboard

juju add-relation keystone mysql

juju add-relation nova-compute mysql
juju add-relation nova-compute rabbitmq-server:amqp
juju add-relation nova-compute nova-cloud-controller
juju add-relation nova-compute glance

juju add-relation nova-cloud-controller mysql
juju add-relation nova-cloud-controller rabbitmq-server
juju add-relation nova-cloud-controller keystone

juju add-relation glance mysql
juju add-relation glance keystone
juju add-relation glance nova-cloud-controller
juju add-relation glance nova-compute

juju add-relation cinder mysql
juju add-relation cinder keystone
juju add-relation cinder rabbitmq-server
juju add-relation cinder nova-cloud-controller
juju add-relation cinder glance

juju add-relation quantum-gateway mysql
juju add-relation quantum-gateway rabbitmq-server
juju add-relation quantum-gateway nova-cloud-controller

juju add-relation openstack-dashboard keystone
```

### Install nuage-nova-compute charm

To install nuage-nova-compute charm, you need to setup a deb repository in the network. MAAS server is an obvious place for this repository. This repository will hold all Nuage packages. Please refer debian documentation (http://debian-handbook.info/browse/stable/sect.setup-apt-package-repository.html) for steps on creating a repository. 
Once the repository is in place create a configuration file with the following contents:

```
nuage-nova-compute:
 vsc-controller-ip: <VSC IP address>
 nuage-repo: deb <repository location> /
```

Copy the nuage-nova-compute charm to the MAAS server under any directory. Deploy the charm with the following commands:

```
juju deploy --config=<nuage-nova-compute.config> --repository=<charm directory> local:precise/nuage-nova-compute
```

The nuage-nova-compute is a subordinate charm to nova-compute which means these two charms get installed in the same container.

You can additional computes by executing "juju add-unit nova-compute". This command will instantiate a new compute with nuage VRS.

Please note that you need to manually install VSC, VSD and Neutron gateway by following the VSP documentation.
