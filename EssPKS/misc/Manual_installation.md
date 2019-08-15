Essential PKS Container cluster on VMware Integrated OpenStack 6.0 Manual Installation
==================================================================
***Follow these instructions only if you wish to replicate the functionality of the HEAT stacks manually.***

Setup Neutron Networking
------------------------

![](../media/VIO%20NCP%20Network%20diagram.png)

a.  ### Create a management Network

    This network will be used to communicate with the kube-apiserver and
    communication between the master and minion nodes.

    Create an API network

b.  ### Create an API network

> This Network will be used to bind the Kubernetes API endpoints and the
> minion nodes will connect to master node via this

c.  ### Create a POD network

    This network will be used for communication between the Kubernetes
    pods.

d.  ### Create an external network

    This network will be used to assign floating IP addresses to the
    cluster nodes.

e.  ### Create a management router

    This router will bridge the management nodes to the floating IP
    addresses

f.  ### Create a container router with SNAT turned off

    This router is used for communications between nodes on the API
    network.

g.  ### Assign the external network as the gateway for both routers

h.  ### Allocate two or more floating IP addresses to your project. You will need as many floating addresses as the number of nodes

i.  ### Setup a security group with rules to allow the following ports:


| Protocol | Port | Purpose |
| :---------------------------------:|:---------------------------------:|:---------------------------------:|
| TCP | 6443\* | Kubernetes API Server |
| TCP | 2379-2380 | etcd server client API |
| TCP | 10250 | Kubelet API |
| TCP | 10251 | kube-scheduler |
| TCP | 10252 | kube-controller-manager |
| TCP | 10255 | Read-Only Kubelet API |
| TCP | 30000-32767 | NodePort Services |

Import Images
-------------

> Import an Ubuntu 18.04 or Ubuntu 16.04 image. NCP is only supported on
> Ubuntu 16.04 and Calico is supported on both 18.04 and 16.04.

Start your Instances
--------------------

a.  ### Create instances for the number of workers + master node

    i.  Select the Ubuntu image you imported earlier.

    ii. Each instance must be on both the management and POD network.

    iii. Pick at least an m1.medium flavor for each node.

    <!-- -->

    a.  ### Designate one node as the master node, you may want to name your node appropriately.

    b.  ### Allocate one floating IP per node for all nodes created.

Download and install required packages on each node
----------------------------------------------------

### Download the Essential PKS packages from:

> <https://downloads.heptio.com/essential-pks/523a448aa3e9a0ef93ff892dceefee0a/vmware-kubernetes-v1.14.0%2Bvmware.1.tar.gz>

### Download the NCP packages from:

> <https://my.vmware.com/web/vmware/details?downloadGroup=NSX-T-PKS-241&productId=673>

c.  ### Copy each package to all the nodes.

d.  ### Install dependent packages

    # apt-get install -y docker.io swapoff

    # systemctl enable docker.service

e.  ### On each node issue the following commands (as root):

    # tar -zxvf vmware-kubernetes-v1.14.0+vmware.1.tar.gz

    # for f in vmware-kubernetes-v1.14.0+vmware.1/kubernetes-v1.14.0+vmware.1/images/*.gz;
    do cat $f | docker load; done
    # swapoff -a

    # dpkg -i vmware-kubernetes-v1.14.0+vmware.1/debs /\*.deb

    # apt-get install -f

    # cd vmware-kubernetes-v1.14.0+vmware.1/kubernetes-v1.14.0+vmware.1/executables/

    # gunzip -c kubeadm-linux-v1.14.0+vmware.1.gz \> /usr/bin/kubeadm

    # gunzip -c kubectl-linux-v1.14.0+vmware.1.gz \> /usr/bin/kubectl

    # gunzip -c kubelet-linux-v1.14.0+vmware.1.gz \> /usr/bin/kubelet

    # chmod +x /usr/bin/{kubeadm,kubelet,kubectl}

If you are using NSX/NCP then do the following steps:
-----------------------------------------------------

a.  ### Install NCP

    # unzip nsx-container-2.4.1.13515827.zip

    # dpkg -i
    nsx-container-2.4.1.13515827/OpenvSwitch/xenial\_amd64/\*.deb

    # apt-get install -f -y

    # systemctl enable openvswitch-switch.service

b.  ### Setup NCP resources and OVS

> Follow the guide to setting up NCP and OVS here, don't apply the cni
> yamls yet:
> <https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.4/com.vmware.nsxt.ncp_kubernetes.doc/GUID-49D7630F-AEB1-4443-BDE0-8788E4415647.html>

c.  ### Set the bridge interface's MAC address to the same as the external interface:

    a.  In /etc/network/interfaces:

        allow-ovs br-int

        iface br-int inet dhcp

        ovs_type OVSBridge

        ovs_ports ens33

        ovs_extra set bridge ${IFACE} other-config:hwaddr=$(ifconfig
        ens33 | grep HWaddr | tr -s " " | cut -d " " -f 5)

    b.  Restart networking

        # service networking restart

Start all services
------------------

### On the master node:

> \# kubeadm init --pod-network-cidr \<your cidr\>
> \--ignore-preflight-errors
>
> \# dpkg -i nsx-container-2.4.1.13515827/Kubernetes/ubuntu\_amd64/
> nsx-cni\_2.4.1.13515827\_amd64.deb
>
> \# docker load -i
> nsx-container-2.4.1.13515827/Kubernetes/nsx-ncp-ubuntu-2.4.1.13515827.tar

### Modify ncp-deployment.yaml and nsx-node-agent.yaml to change the images to the image loaded above.

> \# docker images \| grep ncp
>
> registry.local/2.4.1.13515827/nsx-ncp-ubuntu latest
>
> Change images to "registry.local/2.4.1.13515827/nsx-ncp-ubuntu:latest"

### Start ncp

> \# kubectl apply -f
> nsx-container-2.4.1.13515827/Kubernetes/ubuntu\_amd64/nsx-node-agent-ds.yml
>
> \# kubectl apply -f
> nsx-container-2.4.1.13515827/Kubernetes/ncp-deployment.yml
