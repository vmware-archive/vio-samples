Essential PKS Container cluster on VMware Integrated OpenStack 6.0
==================================================================

VIO 6.0 deployment
------------------

Get a VIO 6.0 deployment up and running, the compute cluster for this
deployment will need enough capacity to start at least two m1.medium VMs
i.e. 4 vCPUs, 8 GB memory and 80Gb storage total.

General Architecture
--------------------

![](../media/VIO%20NCP%20container%20solution.png)

 **Automate Installation with HEAT Stacks**
==========================================

You can automate all the configuration above with this HEAT stack:

EssPKS_1.14.3_NCP_2.5_MP_OS_Cloud_Provider.yaml

**To use this heat stack, you need to do a couple of steps manually**:

**Openstack steps**

1.  Import an Ubuntu 16.04 image:

    You can download the latest Ubuntu cloud image from
    [here](https://cloud-images.ubuntu.com/xenial/current/). Import it
    into your deployment using either Horizon or the OpenStack cli:

2.  Start an instance using the image above.

3.  Download the NCP package from
    [here](https://my.vmware.com/web/vmware/details?downloadGroup=NSX-T-PKS-241&productId=673).
    Copy this image to some permanent location on the instance. Remember
    this location as you'll need it

4.  Take a snapshot of the running instance, you will be using this
    snapshot as the image for the heat stacks.

    **NSX Steps**

<!-- -->

1.  Create an IP block in Advanced Networking & Security IPAM

2.  Create an external IP block in Networking IP Address Management IP
    Address Pools.

    **VIO management steps**

<!-- -->

1.  Get the following TLS certificates from your VIO deployment:

    a.  kubectl -n openstack get secret cinder-tls-public
        -o=jsonpath=\'{.data.tls\\.crt}\' \| base64 --decode \>
        cinder.crt

    b.  kubectl -n openstack get secret nova-tls-public
        -o=jsonpath=\'{.data.tls\\.crt}\' \| base64 --decode \> nova.crt

        **The parameters file for the heat stack:**

parameters:

image: ubuntu\_dual\_nic

key\_name: test

public\_network: public

name: k8scluster

node\_count: 3

nsx\_package\_path: /root/nsx-container-2.5.1.13515827.zip

pod\_network\_cidr: 193.167.0.0/16

kube\_token: yad31n.pil3w87thsvo9img

nsx\_api\_manager: 192.168.111.94

nsx\_username: admin

nsx\_password: Admin!23Admin

tier0\_router: 23fd006e-337a-467a-b3a6-1404a71cc2cf

overlay\_tz: vio-overlay-tz

ip\_block: pod-cidr

external\_ip\_pool: external-ip-pool

os\_username: admin

os\_password: VMware1!

os\_domain\_id: default

os\_tenant\_id: 9a09e26d83854c9cbb321a4e93fd5844

keystone\_ip: 192.168.111.200

keystone\_hostname: keystone.openstack.svc.cluster.local

nova\_ip: 192.168.111.200

nova\_hostnames: nova.vio.local nova.openstack.svc.cluster.local

cinder\_ip: 192.168.111.200

cinder\_hostnames: cinder.vio.local cinder.openstack.svc.cluster.local

**Details on all the parameter options**:

***If you have IP Access disabled (not common) use these parameters*** 

| Parameter                         | value                             |
| --------------------------------- |:---------------------------------:|
| image | Name of the snapshot image containing the NCP package.|
| key_name | Name of the keypair to use for the Kubernetes nodes.|
| public_network | Name of the public network.|
| name | A unique name for the Kubernetes cluster, your node names will be derived with this name and the  NCP ports will be tagged with it as well.|
| node_count | Number of minion nodes to create. Currently this solution is limited to a single master node.|
| nsx\_package_path | Path to the nsx package zip file on the image. This should be a complete filepath.|
| mgmt\_net\_cidr | CIDR for the management network.|
| api\_net\_cidr | CIDR for the API network.|
| nsx\_pod\_net\_cidr  | CIDR for the POD network. This is the network that is used for POD communications, POD IPs themselves are assigned out of the pod\_network\_cidr.|
| pod\_network\_cidr | CIDR for the Kubernetes POD IPs.|
| nameserver | Nameserver IP to use when creating networks and subnets.|
| kube\_token   | A kubeadm token to initialize the master node with. The minions will use this token to join the cluster. This needs to be of the format: \[a-zA-Z\]{6}.\[a-zA-Z0-9\]{16}|
| nsx\_api\_manager | IP address of the NSX API manager.|
| nsx\_username | Username for the NSX API manager.|
| nsx\_password | Password for the NSX API manager.|
| tier0\_router | Name or UUID of the TIER0 router for POD networking.|
| overlay\_tz | Name or UUID of the overlay transport zone to use for the POD networks.|
| ip\_block | Name or UUID of the POD CIDR IP block from NSX.|
| external\_ip\_pool | Name or UUID of the External IP pool from NSX. This pool is used to assign external IP addresses for LoadBalancer type services.|
| os\_username | Openstack username.|
| os\_password | Openstack password.|
| os\_tenant\_id | Openstack Tenant ID.|
| os\_domain\_id | Openstack Domain ID.|
| keystone\_ip  | IP address of the keystone endpoint.|
| keystone\_hostname | Hostname of the keystone endpoint.|
| nova\_ip | IP address of the nova endpoint.|
| nova\_hostname | Hostname(s) of the nova endpoint.|
| cinder\_ip | IP address of the cinder endpoint.|
| cinder\_hostname  | Hostname(s) of the cinder endpoint.|


***\*Copy the downloaded HEAT stack, the parameter file and the nova and
cinder certificates to a common directory. The stack assumes files
called nova.crt and cinder.crt are in the same directory as the stack
file itself. \****

***If you have IP Access enabled (default VIO) use these parameters*** 

| Parameter                         | value                             |
| --------------------------------- |:---------------------------------:|
| image | Name of the snapshot image containing the NCP package.|
| key_name | Name of the keypair to use for the Kubernetes nodes.|
| public_network | Name of the public network.|
| name | A unique name for the Kubernetes cluster, your node names will be derived with this name and the  NCP ports will be tagged with it as well.|
| node_count | Number of minion nodes to create. Currently this solution is limited to a single master node.|
| nsx\_package_path | Path to the nsx package zip file on the image. This should be a complete filepath.|
| mgmt\_net\_cidr | CIDR for the management network.|
| api\_net\_cidr | CIDR for the API network.|
| nsx\_pod\_net\_cidr  | CIDR for the POD network. This is the network that is used for POD communications, POD IPs themselves are assigned out of the pod\_network\_cidr.|
| pod\_network\_cidr | CIDR for the Kubernetes POD IPs.|
| nameserver | Nameserver IP to use when creating networks and subnets.|
| kube\_token   | A kubeadm token to initialize the master node with. The minions will use this token to join the cluster. This needs to be of the format: \[a-zA-Z\]{6}.\[a-zA-Z0-9\]{16}|
| nsx\_api\_manager | IP address of the NSX API manager.|
| nsx\_username | Username for the NSX API manager.|
| nsx\_password | Password for the NSX API manager.|
| tier0\_router | Name or UUID of the TIER0 router for POD networking.|
| overlay\_tz | Name or UUID of the overlay transport zone to use for the POD networks.|
| ip\_block | Name or UUID of the POD CIDR IP block from NSX.|
| external\_ip\_pool | Name or UUID of the External IP pool from NSX. This pool is used to assign external IP addresses for LoadBalancer type services.|
| os\_username | Openstack username.|
| os\_password | Openstack password.|
| os\_tenant\_id | Openstack Tenant ID.|
| os\_domain\_id | Openstack Domain ID.|
| keystone\_ip  | IP address of the keystone endpoint.|

***NOTE: you do not need to supply any hostnames or certificates when IP access is enabled, the HEAT stack will download a certificate.***

***NOTE: The openstack cloud provider uses Public endpoints to communicate with Nova and Neutron, your VM's will need to be able to reach 
         the public endpoints forn these services***

**Create the HEAT stack with**:

\# openstack stack create -e params.yaml \--template
EssPKS\_1.14.3\_NCP\_2.5\_MP\_OS\_Cloud\_provider.yaml k8scluster

*The HEAT stack will do the following:*

1.  Create instances from the image specified

2.  Create the private, API and Container network

3.  Create a router and attach the private network

4.  Create a no-SNAT router for the API network and attach it

5.  Create a POD neutron network.

6.  Set the public network as the gateway for the routers.

7.  Install OVS, docker and Essential PKS packages.

8.  Setup the bridge interfaces on each node

    a.  Create an integration bridge and add the container networking
        interface to it.

    b.  Copy the container interface's Mac address to the bridge br-int

9.  Install the certificates specified.

10. Load all Essential PKS docker images and tag them appropriately.

11. Initialize the master node with the token specified using Kubeadm

12. Initialize the minion nodes to join the master with kubeadm

13. Install NCP Debian packages and load the ncp docker files.

14. Create a default service account with cluster-admin permissions to
    run the nsx node agent and ncp deployment.

15. Tag the appropriate ports in the NSX API manager with cluster and
    node\_name.

16. Add the cloud provider config to all nodes

17. Install the openstack cloud controller manager and the cinder-csi
    plugin pods.

Features
--------

1.  Supports external LoadBalancer via NSX

2.  Supports persistent volume provisioning, attachment and snapshotting
    via Openstack Cinder.
