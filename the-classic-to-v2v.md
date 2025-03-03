---

copyright:
  years:  2021, 2022
lastupdated: "2022-04-28"

keywords: 
content-type: tutorial
services: vpc, virtual-servers
account-plan: paid
completion-time: 60m
subcollection: cloud-infrastructure

---

{:external: target="_blank" .external}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:download: .download}
{:important: .important}
{:note: .note}
{:step: data-tutorial-type='step'}

# Classic VMware VM to IBM Cloud VPC migration with RMM  
{: #migrating-images-vmware-vpc-classic}
{: toc-content-type="tutorial"} 
{: toc-services="vpc, virtual-servers"} 
{: toc-completion-time="60m"}

The RackWare Management Module (RMM) migration solution provides a seamless virtual-to-virtual replatforming for VMware virtual machine (VM) to {{site.data.keyword.cloud}} virtual server instance migration. It allows the adoption of existing capabilities of {{site.data.keyword.cloud_notm}}. Its intuitive GUI permits moving the OS, application, and data from VMware ESXi in {{site.data.keyword.cloud_notm}} classic to {{site.data.keyword.vpc_short}} virtual server instances.
{: shortdesc}
 
This guide shows you how to complete a migration from VMware ESXi in {{site.data.keyword.cloud_notm}} classic to {{site.data.keyword.vpc_short}}.

## Supported operating systems
{: #supported-operating-systems-classic}

- CentOS 7.8, 7.9
- RHEL 7.2, 7.3, 7.4, 8.1 
- Ubuntu 18.04, 20.04
- Debian 9.x, 10.x 
- Windows 2012, 2012R2, 2016, 2019 

## Architecture diagram
{: #architecture-diagram-classic}

![Topology](images/Classic.svg){: caption="Figure 1. Architecture diagram" caption-side="bottom"}
 
## Before you begin
{: #before-begin-classic}

* Check for correct {{site.data.keyword.vpc_short}} user permissions. Be sure that your user account has sufficient permissions to create and manage VPC resources. See the list of [required permissions](/docs/vpc?topic=vpc-managing-user-permissions-for-vpc-resources) for VPC.
* Understand the capability differences between VMware and VPC infrastructures (the following list is not exhaustive):
    * VPC does not support shared volumes or file-based volumes
    * GPU is not supported in VPC
    * Encrypted volumes are not supported

To improve data transfer rate, adjust the bandwidth allocation of the RMM server. For more information, see [Adjusting bandwidth allocation using the UI](/docs/vpc?topic=vpc-managing-virtual-server-instances&interface=ui#adjusting-bandwidth-allocation-ui).   
{: note}

## Order RMM
{: #order-rackware-rmm-classic}
{: step}

The RMM tool is available in the {{site.data.keyword.cloud_notm}} catalog. After you order, a virtual server with RMM software is installed into your VPC of choice. The RMM server has a public IP address for reachability and a default login.

1. Order the RMM server from the [{{site.data.keyword.cloud_notm}} catalog](https://cloud.ibm.com/catalog/content/IBM-MarketPlace-P2P-1.3-22935832-bd76-49ab-b53e-12fc5d04c266-global){: external}.

2. After you order, log in to the RMM server.

3. In the RMM server, change the default password, create users, and create an SSH key.

4. Upload the SSH key to {{site.data.keyword.vpc_short}}.
 
## Bring Your Own License (BYOL) from RackWare
{: #license-rackware-bring-classic}
{: step}

1. Generate a license file in `/etc/rackware` by running the following command:                    

   ```
   rwadm relicense 
   ```
   {: pre}
 
2. You need to purchase the license from RackWare by emailing the generated license file to [licensing@rackwareinc.com](mailto:licensing@rackwareinc.com) or [sales@rackwareinc.com](mailto:sales@rackwareinc.com). 

3. After you receive a valid license, download the license file and place it in `/etc/rackware`. Restart the services to apply the license by running the following command:
 
   ```
   rwadm restart 
   ```
   {: pre}
 
4. Verify the license by running the following command:
   
   ```
   rw rmm show 
   ```
   {: pre}

## (Optional) Transit Gateway for private connectivity
{: #connectivity-private-vpc}
{: step}

The {{site.data.keyword.tg_full_notm}} provides connectivity between VMware {{site.data.keyword.cloud_notm}} classic and your VPC infrastructure. 

When you connect the two entities, be aware that the VMware classic infrastructure is on the 10.0.0.0 network and thus the VPC network needs to reside on a different network. Otherwise, the network overlaps and there is a communication connectivity issue.
{: important}
 
1. Order {{site.data.keyword.tg_full_notm}}

    a. Use local routing option 

2. Add connections

    a. Classic infrastructure 

    b. VPC 

    - Select region 

## Set up and provision VPC and virtual server instance
{: #cloud-vpc-vsi-setup}
{: step}

The RMM solution handles only the OS, application, and data movement. It does not to set up a VPC target side; you need to handle this. You will first need to set up the VPC infrastructure. At a bare minimum, you need to set up a VPC, subnets, and the corresponding instances that you are planning to migrate. The new target virtual server instance profile (vCPU and vMemory) does not need to match the source. However, as for the storage, it needs to be the same or greater in size. This guide does not go through all of the details for setting up the VPC infrastructure. For more information, see [Getting started with Virtual Private Cloud (VPC)](/docs/vpc?topic=vpc-getting-started).

1. Create a VPC.

2. Create subnets.

3. Order the virtual server instance:

    a. SSH key (RMM SSH keys need to be added in addition to bastion SSH key)

    b. OS name (same major version as the source)

    c. Security groups

    d. Secondary volume (optional)

Ensure that your VPC, subnet, and other necessary cloud components are set up before adding the cloud user in RMM.
{: note}

## Source and target compute preparation
{: #source-target-compute-prep-vmware}
{: step}

Before you start the migration, RMM server needs to SSH into the servers. Thus, the RMM public SSH key needs to be copied on both the source and target servers. 
 
For Windows OS, you need to download the SSH key utility. You can download it from RMM server. 
{: note}
 
For Windows OS, the user is `SYSTEM` and you must key in the RMM SSH key here to authenticate for both source and target servers.
{: note}

## Set up RMM waves
{: #rackware-rmm-migration-classic}
{: step}

You can migrate the servers one-by-one or opt to perform multiple simultaneous migrations. If you are performing multiple simultaneous migrations, then download the CSV template from the RMM server and complete the appropriate fields.

1. Log in to the RMM server.
2. Create a _Wave_ and define _Wave_ name.
3. If there are multiple hosts, download the template, complete the appropriate fields, and then upload the template.
4. Select the _Wave_ name to enter source and target information.
5. Select the "+" sign.
6. Add source IP address or FQDN and add source username. 
7. Target Type = Existing system
8. Sync Type = Direct sync
9. Add target IP address or FQDN.
10. Add a target-friendly name, and add a target username.
11. Start the migration.
 
The username field for the Linux environment is `root`. The username field for the Windows environment is `SYSTEM`.
{: note}
 
Alternatively, you can use the discovery helper script, which helps with the discovery of virtual machines on the VMware ESXI Host and also creates corresponding waves on the RMM server. The script asks for your vSphere host username and for the IP address of the vSphere to connect to and the API where you discover your on-premises classic VMware ESXi Host VMs. 
 
```
./discoveryTool -s <vSphere> -u <username of the Vspherehost>
```
{: pre}

Example:

```
$ ./discoveryTool -s 10.10.10.9 -u administrator@vsphere.local
```
{: screen}  
 
For more information about discovering the VMware guest VMs using the discovery tool, see this [public GitHub repository](https://github.com/IBM-Cloud/vpc-migration-tools/blob/main/v2v-discovery-tool-rmm/VMware/README.md){: external}

## Validate your migration
{: #rackware-rmm-validation-classic}

Before you decommission the source server, it is imperative to validate the target server. This is not an exhaustive list but some of the items to validate are:

- Application 
- Licensing 
- Reachability (host level configuration changes) 
- Remove RMM SSH key after migration successful 
 
## More resources
{: #rackware-rmm-resources-classic}

1. [Discovery Tool](https://github.com/IBM-Cloud/vpc-migration-tools/blob/main/v2v-discovery-tool-rmm/VMware/README.md){: external}
2. [FAQs](/docs/cloud-infrastructure?topic=cloud-infrastructure-faqs-vmware)  
3. [RackWare usage guide](https://www.rackwareinc.com/cloud-migration){: external}
