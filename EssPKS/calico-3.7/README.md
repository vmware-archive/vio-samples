Essential PKS with Calico CNI
=============================

This setup is quite a bit simpler than with NCP and requires no manual
intervention at any point. To bring it up follow the steps from above to
setup neutron networking with the following changes:

-   You don't need a separate POD network and a second router

![](media/image1.png){width="6.278480971128609in"
height="7.233517060367454in"}

Use the following Heat stack to install Essential PKS 1.14.3, Calico 3.7
and OpenStack cloud provider:

[Essential PKS Calico Cloud
Provider](https://onevmw-my.sharepoint.com/:u:/g/personal/asomya_vmware_com/ERw4IqSrlThFjEd71y9uRCkB0Lspmma1kYqTkhEJSx7oCA?e=e4UHAH)

This stack will install Essential PKS 1.14.3, Calico 3.7 and initialize
the Openstack Cloud Provider. It requires the following parameters to be
passed:

Parameters:
-----------

  image                 Name of the ubuntu image to use
  --------------------- ----------------------------------------------------------------------------------------------------------
  key\_name             Name of the keypair
  public\_network\_id   UUID of the public/floating network
  public\_subnet\_id    UUID of the public/floating subnet
  name                  A unique name for your cluster, the nodes will be named after this name
  pod\_network\_cidr    The POD network CIDR
  kube\_token           A token for Kubernetes nodes to join the master. It should be in the format \[a-z\]{6}.\[a-zA-Z0-9\]{16}
  os\_username          Openstack username
  os\_password          Openstack password
  os\_domain\_id        Openstack domain uuid
  os\_tenant\_id        Openstack tenant/project uuid
  keystone\_ip          Keystone endpoint IP address
  keystone\_hostname    Keystone hostname
  username              A user to add to the node
  password              Password for the node user
  nova\_ip              Nova endpoint IP address
  nova\_hostnames       Hostname(s) for nova
  neutron\_ip           Neutron endpoint IP address
  neutron\_hostnames    Hostname(s) for neutron

### Example Parameter file:

parameters:

image: ubuntu\_dual\_nic

key\_name: test

public\_network\_id: ddd59d0e-d1b7-4bc5-b9c5-7f11e5523813

public\_subnet\_id: f6602cbe-b771-4e10-9457-fa98c047c38d

name: k8scluster

pod\_network\_cidr: 192.167.0.0/16

kube\_token: yad31n.pil3w87thsvo9img

os\_username: admin

os\_password: VMware1!

os\_domain\_id: default

os\_tenant\_id: 9a09e26d83854c9cbb321a4e93fd5844

keystone\_ip: 192.168.111.200

keystone\_hostname: keystone.openstack.svc.cluster.local

username: vmware

password: vmware

nova\_ip: 192.168.111.200

nova\_hostnames: nova.vio.local nova.openstack.svc.cluster.local

neutron\_ip: 192.168.111.200

neutron\_hostnames: neutron.vio.local
neutron.openstack.svc.cluster.local

**You will also need to get certificates for nova, neutron and cinder
from your VIO deployment using:**

\# kubectl -n openstack get secret cinder-tls-public
-o=jsonpath=\'{.data.tls\\.crt}\' \| base64 --decode \> cinder.crt

\# kubectl -n openstack get secret nova-tls-public
-o=jsonpath=\'{.data.tls\\.crt}\' \| base64 --decode \> nova.crt

\# kubectl -n openstack get secret neutron-tls-public
-o=jsonpath=\'{.data.tls\\.crt}\' \| base64 --decode \> neutron.crt

The heat stack assumes these three certificates named nova.crt,
neutron.crt and cinder.crt are present in the same directory as the Heat
stack.

**Create the HEAT stack with**:

\# openstack stack create -e params.yaml \--template
EssPKS\_1.14.3\_Calico\_3.7\_OS\_Cloud\_provider.yaml k8scluster

**The Heat stack will get you the following**:

-   Essential PKS 1.14.3

-   Calico CNI 3.7

-   Openstack cloud provider config

-   LoadBalancer support out of the box with the supplied public network
    and subnet id

-   Openstack cloud provider

-   Cinder CSI plugin for persistent volumes

-   Neutron LBaaS v2 support for LoadBalancer type services.
