Essential PKS with Calico CNI
=============================

This setup is quite a bit simpler than with NCP and requires no manual
intervention at any point.

Use the following Heat stack to install Essential PKS 1.14.3, Calico 3.7
and OpenStack cloud provider:
```
EssPKS_1.14.3_Calico_3.7_OS_Cloud_Provider_IP_Access.yaml
```
This stack will install Essential PKS 1.14.3, Calico 3.7 and initialize
the Openstack Cloud Provider. It requires the following parameters to be
passed:

Features
--------

1.  Supports native LoadBalancer via Neutron LBaasV2.

2.  Supports persistent volume provisioning, attachment and snapshotting via Openstack Cinder.

3. Supports multiple master nodes for High Availability.

4. User can scale the deployment up or down depending on requirements and resource availability.

Parameters:
-----------

| Parameter | Value|
|:---:|:-----|
|  image | Name of the ubuntu image to use |
| key\_name | Name of the keypair |
| master_count | Number of master nodes to create.|
| minion_count | Number of minion nodes to create.|
| public\_network\_id | UUID of the public/floating network |
| public\_subnet\_id | UUID of the public/floating subnet |
| name | A unique name for your cluster, the nodes will be named after this name |
| pod\_network\_cidr | The POD network CIDR |
| os\_username | Openstack username |
| os\_password | Openstack password |
| os\_domain\_id | Openstack domain UUID |
| os\_tenant\_id | Openstack tenant/project UUID |
| keystone\_ip | Keystone endpoint IP address |
| ssh\_private\_key  | Private key to use for the nodes to ssh to each other.|

### Example Parameter file:

```
parameters:

  image: ubuntu_dual_nic

  key_name: test

  public_network_id: ddd59d0e-d1b7-4bc5-b9c5-7f11e5523813

  public_subnet_id: f6602cbe-b771-4e10-9457-fa98c047c38d

  name: k8scluster

  pod_network_cidr: 192.167.0.0/16

  os_username: admin

  os_password: VMware1!

  os_domain_id: default

  os_tenant_id: 9a09e26d83854c9cbb321a4e93fd5844

  keystone_ip: 192.168.111.200

```

**Create the HEAT stack with**:
```
# openstack stack create -e params.yaml --template vio.heatstacks/EssPKS/calico-3.7/EssPKS_1.14.3_Calico_3.7_OS_Cloud_provider_IP_access.yaml \
--parameter-file ssh_private_key=/path/to/my/private/key/file k8scluster
```

**The Heat stack will get you the following**:

-   Essential PKS 1.14.3

-   Calico CNI 3.7

-   Openstack cloud provider config

-   LoadBalancer support out of the box with the supplied public network and subnet id

-   Openstack cloud provider

-   Cinder CSI plugin for persistent volumes

-   Neutron LBaaS v2 support for LoadBalancer type services.

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
openstack --insecure stack update -e params.yaml --template vio.heatstacks/EssPKS/calico-3.7/EssPKS_1.14.3_Calico_3.7_OS_Cloud_provider_IP_access.yaml \
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
