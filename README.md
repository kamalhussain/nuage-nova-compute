Nuage-nova-compute Charm
==========================
This charm provides VRS functionality on an existing nova-compute. The nuage-nova-compute charm is a subordinate to nova-compute which means both charms get installed in the same container. Before installing this charm, the OpenStack cluser with nova-compute should have been installed. Please follow instructions at https://github.com/kamalhussain/nuage-nova-compute/blob/master/OpenStack-Nuage.md to make this happen.

Usage
------------------
First, copy the charm to the MAAS server under any directory

Second, create a configuration file with the following contents:

```
nuage-nova-compute:
 vsc-controller-ip: <ip address>
 nuage-repo: deb <apt repository> /
```

Finally, deploy the charm 

```
juju deploy --config=<config file> --repository=<charm directory> local:<directory>/nuage-nova-compute
```


