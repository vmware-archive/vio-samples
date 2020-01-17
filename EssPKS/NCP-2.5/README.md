<!--
Copyright © 2019 VMware, Inc. All Rights Reserved.
SPDX-License-Identifier: BSD-2-Clause
-->
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

Networking details
------------------

![](../media/VIO%20NCP%20Network%20diagram.png)

 **Automate Installation with HEAT Stacks**
 ==========================================

 To bring up an Essential PKS Cluster with NCP 2.5 Management Plane on top of VIO 6.0 use the following HEAT Stack:

 _EssPKS_1.14.3_NCP_2.5_MP_OS_Cloud_Provider.yaml_

 **To use this heat stack, you need to do a couple of steps manually**:

 **Openstack steps**

 1.  Import an Ubuntu 16.04 image:

     You can download the latest Ubuntu cloud image from
     [here](https://cloud-images.ubuntu.com/xenial/current/). Import it
     into your deployment using either Horizon or the OpenStack cli:

 2.  Start an instance using the image above.

 3.  Download the NCP package from
     [here](<Link to NCP 2.5 download url>).
     Copy this image to some permanent location on the instance. Remember
     this location as you'll need it

 4.  Take a snapshot of the running instance, you will be using this
     snapshot as the image for the heat stacks.

 **NSX Steps**

 <!-- -->

 1.  Create an IP block in Advanced Networking & Security --> IPAM

 2.  Create an external IP block in Networking --> IP Address Management IP -- > Address Pools.

 **VIO management steps (Only for Ingress based deployments)**

 ***The parameters file for the heat stack:***

 ```
 parameters:

   image: ubuntu_dual_nic

   key_name: test

   public_network: public

   name: k8scluster

   master_count: 2

   minion_count: 3

   nsx_package_path: /root/nsx-container-2.5.123456.zip

   pod_network_cidr: 193.167.0.0/16

   nsx_api_manager: 192.168.111.94

   nsx_username: admin

   nsx_password: Admin!23Admin

   tier0_router: 23fd006e-337a-467a-b3a6-1404a71cc2cf

   overlay_tz: vio-overlay-tz

   ip_block: pod-cidr

   external_ip_pool: external-ip-pool

   os_username: admin

   os_password: VMware1!

   os_domain_id: default

   os_tenant_id: 9a09e26d83854c9cbb321a4e93fd5844

   keystone_ip: 192.168.111.200

 ```
 **Details on all the parameter options**:

 ***If you have IP Access enabled (default VIO) use these parameters***

 | Parameter                         | Value                             |
 | --------------------------------- |:---------------------------------|
 | image | Name of the snapshot image containing the NCP package.|
 | key_name | Name of the keypair to use for the Kubernetes nodes.|
 | public_network | Name of the public network.|
 | name | A unique name for the Kubernetes cluster, your node names will be derived with this name and the  NCP ports will be tagged with it as well.|
 | master_count | Number of master nodes to create.|
 | minion_count | Number of minion nodes to create.|
 | master_flavor | Instance flavor for master nodes. default: m1.medium |
 | minion_flavor | Instance flavor for minion nodes. default: m1.medium |
 | nsx\_package_path | Path to the nsx package zip file on the image. This should be a complete filepath.|
 | mgmt\_net\_cidr | CIDR for the management network.|
 | api\_net\_cidr | CIDR for the API network.|
 | nsx\_pod\_net\_cidr  | CIDR for the POD network. This is the network that is used for POD communications, POD IPs themselves are assigned out of the pod\_network\_cidr.|
 | pod\_network\_cidr | CIDR for the Kubernetes POD IPs.|
 | nameserver | Nameserver IP to use when creating networks and subnets.|
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
 | ssh\_private\_key  | Private key to use for the nodes to ssh to each other.|

 **The Essential PKS 1.16 NCP stack has five additional options for SRIOV networks:**

 | Parameter                         | Value                             |
 | --------------------------------- |:---------------------------------|
 | install_nad_crd  | Install Custom Resource Definition for a NetworkAttachmentDefinition.|
 | sriov_provider_net  | Name or Id of the SRIOV provider network to reserve SRIOV ports on.|
 | sriov_subnet  | Name or Id of the SRIOV subnet to reserve SRIOV ports on.|
 | sriov_ports_to_reserve  | Number of SRIOV ports to reserve on each minion node.|
 | sriov_interface_number_start  | The HEAT stack will rename your SRIOV interfaces for consistency across installations. You can specify the number to start naming the interfaces, it is 100 by default. The interfaces will be named ens 100, ens101 an so on. |
 **NOTE:**

 ***The openstack cloud provider uses Public endpoints to communicate with Nova and Neutron, your VM's will need to be able to reach
 the public endpoints forn these services***

 ***Use the private key from the keypair used to create the nodes for the ssh_private_key parameter.***

 **Create the HEAT stack with**:

 ```
 openstack --insecure stack create -e params.yaml -t vio.heatstacks/EssPKS/NCP-2.4/EssPKS_1.14.3_NCP_2.4_OS_Cloud_provider_IP_Access.yaml \
 --parameter-file ssh_private_key=/path/to/my/private/key/file k8scluster
 ```

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
     node_name.

 16. Add the cloud provider config to all nodes

 17. Install the openstack cloud controller manager and the cinder-csi
     plugin pods.

 Testing your deployment
 =======================
 1. Check if all nodes are up and running and have joined the cluster successfully
    * SSH to your main master node, you will see a node named <cluster-name>-master-main

    * On the main master node execute:
 ~~~~
 kubectl get no -o wide
 ~~~~
   * Verify all master nodes have internal and external IP addresses
 ```
 NAME                     STATUS   ROLES    AGE     VERSION            INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
 k8scluster-master-0      Ready    master   3d23h   v1.14.3+vmware.1   11.0.0.76     5.5.5.39      Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.9.7
 k8scluster-master-main   Ready    master   3d23h   v1.14.3+vmware.1   11.0.0.252    5.5.5.197     Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.9.7
 k8scluster-minion-0      Ready    <none>   3d23h   v1.14.3+vmware.1   11.0.0.29     <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.9.7
 k8scluster-minion-1      Ready    <none>   3d23h   v1.14.3+vmware.1   11.0.0.250    <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.9.7
 ```

 2. Deploy a sample from the examples to test provisioning.
    * Deploy the examples/nginx_ncp_csi_pv_nsx_lb.yaml file using
 ~~~~
 kubectl apply -f nginx_ncp_csi_pv_nsx_lb.yaml
 ~~~~
    * Verify a cinder volume was provisioned and attached to a node in Openstack
    * Verify the persistent volume was created and bound successfully in Kubernetes
 ```
 root@k8scluster-master-main:~# kubectl get pvc
 NAME                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
 csi-pvc-cinderplugin   Bound    pvc-fdb869e9-ce73-11e9-8866-fa163e20a3a3   1Gi        RWO            csi-sc-cinderplugin   30s
 root@k8scluster-master-main:~# kubectl get pv
 NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                          STORAGECLASS          REASON   AGE
 pvc-fdb869e9-ce73-11e9-8866-fa163e20a3a3   1Gi        RWO            Delete           Bound    default/csi-pvc-cinderplugin   csi-sc-cinderplugin            40s
 root@k8scluster-master-main:~#
 ```

 Scaling your deployment
 =======================

 To scale up your deployment adjust these two parameters in your params.yaml file, you can increase or decrease the
 number of nodes of each type, you still need a minimum of 1 master node and 1 minion node:
 ~~~~
 minion_count: 2
 master_count: 2
 ~~~~
 Increase or decrease the number to the desired number of master and/or minion nodes and update the stack with
 ```
 openstack --insecure stack update -e params.yaml --template vio.heatstacks/EssPKS/NCP-2.4/EssPKS_1.14.3_NCP_2.4_OS_Cloud_provider_IP_Access.yaml \
 --parameter-file ssh_private_key=/path/to/my/private/key/file  k8scluster
 ```

 Keep the rest of the parameters the same as when the stack was created.

 Verify nodes were created or deleted based on your desired node count by running the following command on any master node:
 ~~~~
 kubectl get no -o wide
 ~~~~

 External Access to the Cluster
 ==============================

 The heat stacks create a Neutron LoadBalancer on the management network with a FloatingIP from the public network
 specified in the parameters file. The kubeconfig on each master node uses this IP address as the Kubernetes API endpoint.

 *You can get the kubeconfig file from (on any master node):*
 ```
 /etc/kubernetes/admin.conf
 ```
