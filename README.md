<!--
Copyright © 2019 VMware, Inc. All Rights Reserved.
SPDX-License-Identifier: BSD-2-Clause
-->
# VMware has ended active development of this project, this repository will no longer be updated.
# VIO Samples
# Overview

This repository contains example codes or templates for VMware Integrated OpenStack (VIO). 

#### Structure:

**EssPKS**/ : This directory contains HEAT stack templates to instantiate an Essential PKS cluster on VIO 6.0.

&nbsp;&nbsp;  **calico-3.7**: HEAT stacks with the Calico 3-7 CNI plugin<br>
&nbsp;&nbsp;  **examples**: Sample files<br>
&nbsp;&nbsp;  **lib**: Library modules used by the stacks<br>
&nbsp;&nbsp;  **media**: Supporting files for README files e.g. diagrams<br>
&nbsp;&nbsp;  **misc**: Miscellaneous items, not essential<br>
&nbsp;&nbsp;  **NCP-2.4**: HEAT stacks with the NCP 2.4 CNI plugin<br>
&nbsp;&nbsp;  **NCP-2.5**: HEAT stacks with the NCP 2.5 CNI plugin<br>


Please refer to the README files for specific stacks for details and installation instructions.

### Prerequisites

* Vmware Integrated Openstack (VIO) 6.0
* NSX-T 2.4/2.5
* Compute capacity to start at least two master and two minion nodes with a minimum m1.medium flavor.

## Contributing

The vio-samples project team welcomes contributions from the community. Before you start working with vio-samples, please
read our [Developer Certificate of Origin](https://cla.vmware.com/dco). All contributions to this repository must be
signed as described on that page. Your signature certifies that you wrote the patch or have the right to pass it on
as an open-source patch. For more detailed information, refer to [CONTRIBUTING.md](CONTRIBUTING.md).
