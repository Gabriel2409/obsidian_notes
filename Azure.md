#tuto #todo

## Azure AD

- `az login`
- OR go to `>Azure Active Directory`

Azure AD is a managed cloud based identity and directory management service enabling access to Azure services and other SaaS solutions

### Concepts

- Identity = any object that can be authenticated (user, group, managed identity, service principals)
- Account = Identity + data attributes
- Azure AD account = accounts created in Azure AD
- Azure AD tenant or directory = Dedicated instance of Azure AD created during sign up of any subscription. Note that tenants can have multiple subscriptions

### User accounts

Click on `Manage|Users`

- Type of users:
  - Cloud Identities = users that only exist in azure AD
  - Guest Accounts = users that exist outside of azure that can be invited (microsoft accounts for ex)
  - Directory Synced Identities = synchronized from on permises Windows AD. Can not be created
- Possibility to do bulk operations by uploading a csv

### Groups

Click on `Manage|Groups`
Groups can be used for easier set up of RBAC. Users that are part of a group inherit its permissions

- Type of groups:

  - Security Group: standard groups
  - Microsoft 365 groups: asssign a shared mailbox, calendar, sharepoint site, etc

- Membership type:
  - Assigned: add manually users to a group
  - Dynamic User: join group based on user attributes
  - Dynamic Device: join group based on device attributes

### Azure AD Join

- Make sure devices are secure and follow corporate rules
- offers SSO, device management, access to on-prems apps, etc

### Self service password reset (SSPR)

Click on `Manage>Users>Password reset` (premium feature)

- set up multiple methods to reset password without having to go through IT help desk

### Multi tenant environment

Go to `Overview_Manage tenants`
Each tenant is fully independant: no parent-child relationship

## Subscriptions and Governance

### Managing Subscriptions

- logical container that defines billing boundary and usage (has a unique id)
- Deployed Resources are mapped to a subscription
- An account can have multiple subscription

Note: tenant vs subscription
**A tenant is an Azure Active Directory (Azure AD) entity. It’s the directory in which users, groups, and applications are stored. A subscription is a billing entity that you use to organize access to cloud resources. You can have multiple subscriptions in a single tenant.**

### Hierarchy

RBAC, policy management logs, etc... can be defined at several levels:

- Management groups: scope above subscripton. There is a root management group that can contain several management groups
- Subscriptions: By default, they are part of the root management group, but we can choose which management group they are part of
- Resource groups
- Resources

Each scope inherits the permission of its parents.

### RBAC

Enables admins to grant acces to azure resources

- Who: Security Principal: any identity requesting access
- What: Role definition: set of operation a role can perform (written in JSON)
- Where: Scope: limit of access, boundaries
  Combination of 3: role definition atached to a service principal and a scope) = **role assignment** (max 2000 per subscription)

Azure RBAC:

- Used to manage access to azure resources
- scope include management groups, subscriptions, resource groups and resources
- example of roles: owner, contributor, reader,...

Azure AD Roles:

- Used to manage azure AD features
- scope is at the tenant level
- example roles: Global Administrator, Billing Administrator

Be sure to follow the principle of least privileges

### Custom and built in roles

4 main built-in roles to remember in Azure RBAC:

- owner: full access to resource + can delegate access
- contributor: same as owner but can not grant access
- reader: read access to all resources
- user access administrator: can give access to resources

Custom RBAC roles:

- needed to fine tune roles
- must define associated scope

### Azure tags

- add metadata to subscription, resource groups, resources
- key value pairs allowing logical grouping : can be used to filter azure usage and cost management
- Tags are not inherited. If you want to use them for cost management, they must be there at the resource level, not the resource group level

### Resource locks

- protect resources from accidental change or deletion
- Locks support inheritance
- Two types:
  - Read only locks: prevents any change to the resource
  - Delete locks: resources can be modified but not deleted

### Analyzing costs

- You can save costs with Azure Reserved Instances (RI)
- Every region has a different price

### Azure policies

Policies can be used to define organizational standards and identify non-compliant resources

- Definition: JSON Document to describe the effect
- Scope: Where we enforce the policy
- Assignment: assign a policy to a scope
- Complicance: evaluate the compliance and understand non compliant resources

Example of policies:

- Allowed resources types
- Allowed virtual machines
- Allowed locations
- Enforce tags that need to be added
- Inherit tags from subscription or resource group (by default it is not the case)
- Allowed resource group locations

Initiatives: group policies together

## Virtual network

### Creating and configuring virtual networks

Go to `>Virtual networks`

- Each VNet is associated to a Subscription and is created within a region
- VNet has an address space (can be public or private) => do not let the address space overlap with you on-premises resources. When a resource is created in the VNet, the IP address is given from the address space
- The VNet contains subnets, for ex GatewaySubnet, databaseSubnet, etc.., each getting an IP address from the address allocated to the subnet

### Private and public IP addresses

- Private IP addresses
  - Used withing Azure Virtual Network, and with hybrid scenarios involving VPN gateways and ExpressRoute connections
  - Allocations methods
    - Static: for domain controllers, web servers and DNS servers, ... which do not change even if servers are rebooted
    - Dynamic: defaut option, address is dynamically allocated from the address pool
- Public IP addresses:
  - Used to communicate with the Internet and other public facing services (Azure databases, storage accounts,...)
  - Also comes in 2 allocations types: static and dynamics
  - SKU Basic vs standard

Public IP addresses must be created as separate resources and then allocated to a given resource in azure. For ex, you can give a public IP to a VM to allow to connect to it via SSH

### User Defined Routes

System routes allow us to

- Communicate between VMs in the same subnet
- Communicate between VMs in different subnets in the same network
- Communicate from VM to Internet (No need to configure a NAT gateway like in other providers. It is blocked by defautl though)
- Communicate via Site-to-Site and ExpressRoute connection while using VPN gateways

Route tables allow us to create User Defined Routes and overwrite system routes to reroute the traffic

### Service endpoints

By default, to make our Vnet communicate with other services, we must use Public IP, which may become problematic if the allocation is dynamic. We can switch to private IP by using Service Endpoint, which means we don't need to maintain a list of IP addresses.

- Benefits:
  - Access azure services with better security (no transit via internet)
  - Leverage Microsoft backbone network (private network owned by microsoft)
  - Most services are supported

Example:

- create a vnet
- create a vm
- create a storage account
  - create a container allowing anonymous access
  - add a file in the container
  - In the networking tab, select `Enabled from selected virtual networks and IP addresses` and add the vnet => blobs in the container are no more available (you can not even access the viewer from azure)
- ssh into the VM
  - check you can get a file from the storage account: `wget ...`
  - run `dig <mystorage>.blob.core.windows.net`
    - In the answer section, you can see that the IP is a PUBLIC IP address

Note: the service endpoint can be seen in the vnet.

### Private links

With service endpoints, we are using private IP addresses but the destination is public.
We can instead use private links to make resources not in the same VNet communicate
Private links are one to one mapping and are truly private.

Example:

- take the storage from before and remove the service endpoint
- In `Networking_Private endpoints`, create a new private endpoint
- in the vm, try to `wget` again: it should resolve to the private IP address

Note: the private link can be seen in the vnet.

### Azure DNS

- Host DNS zones for name resolution
- Create records (name must be unique within a resource group)

Ex:

- create a new dns zone
- Add an A record
- Run `nslookup <record-name>.<dns-name> <azure-server>` or `dig @<azure-server> <record-name>.<dns-name>` (alternatively download DNSDataView)

### Private zones

Azure dns are public by default. We can also create private dns zones.
For ex, if you have vms in two different subnets, we could create a private DNS zone with autoregistration which would automatically map each vm name to their IP address.
To do that, we need to create 2 virtual network link (one to each network).
Then any vm could get the IP of any other VM by using its name and the private DNS zone. Note that it does not allow a VM from a subnet to connect to a VM from another subnet because communication is not enabled

### Network security groups NSG

- NSG operates at the layer 4 and allows to filter incoming and outgoing traffic from a vnet
- Rules have an associated priority
- NSG can be associated to subnets and Network interface
- Rules are evaluated at the subnet and the NI separately

Inbound traffic: source -> Subnet NSG -> NI NSG
Outbound traffic: VM -> NI NSG -> Subnet NSG

At the NI level: go to the vm then networking
By default no NSG is attached to the subnet

### Azure Firewall

- Highly available and scalable
- Redundant
- Multiple types of rules
- Threat intelligence
- Public IP support

Should be in the AzureFirewallSubnet and can be connected to any nb of Vnets.

Can become the central point for all traffic with the help of route table.
Ex:

- Create a new route:

  - name: force-to-fw
  - Destination Address prefix = IP addresses
  - Destination IP Addresses / CIDR ranges: 0.0.0.0/0
  - Next hop type: Virtual appliance
  - Next hop address: provide the PRIVATE ip of azure fw

- Then associate the chosen subnet (not the AzureFirewall subnet). Any traffic going through this subnet will use the route table. Note that here, the subnet and AzureFirewall are part of the same VNet. I think that if we want to use AzureFirewall in a different vnet, we must add private dns zones to make it work
- To connect via ssh to a vm, Add NAT rule collection where destination address is azure firewall PUBLIC ip and translated address is vm PRIVATE ip. Then to connect specify the address of the firewall not the vm

## Configure VMs

### Planning VMs

Shared responsibility model defines the responsibility shared with the cloud provider:

- Responsibility transfered to cloud provider:
  - physical hosts
  - physical network
  - physical datacenter
- Responsibility varies depending on service type (IaaS, PaaS, SaaS)
  - Operating system
  - Network controls
  - Applications
  - Identity and directory infra structure
- Responsibility always retained by customer
  - Information and data
  - Devices
  - Accounts and identities

With VMs, we are using IaaS, which means we are reponsible for everything that is not physical. Virtual machines require planning:

- Networking: we need to plan our networking address spaces based on the number of virtual machines and make sure there is no address overlap if there are multiple networks
- Naming: naming convention are very important. For ex a prod web server in east us could be named web-prod-eus
- Location: choose low cost regions if possible and choose regions chloser to customers to avoid performance issues.
- Pricing: consider models such as Pay as you go and Reserved Instances. For low priority dev workload, use spot vms.

### Managing VM sizes

https://learn.microsoft.com/en-us/azure/virtual-machines/sizes-general

| General purpose          | B, Dsv3, Dv3, Dasv4, Dav4, DSv2, Dv2, Av2, DC, DCv2, Dpdsv5, Dpldsv5, Dpsv5, Dplsv5, Dv4, Dsv4, Ddv4, Ddsv4, Dv5, Dsv5, Ddv5, Ddsv5, Dasv5, Dadsv5 | Balanced CPU-to-memory ratio. Ideal for testing and development, small to medium databases, and low to medium traffic web servers.                                                              |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Compute optimized        | F, Fs, Fsv2, FX                                                                                                                                    | High CPU-to-memory ratio. Good for medium traffic web servers, network appliances, batch processes, and application servers.                                                                    |
| Memory optimized         | Esv3, Ev3, Easv4, Eav4, Epdsv5, Epsv5, Ev4, Esv4, Edv4, Edsv4, Ev5, Esv5, Edv5, Edsv5, Easv5, Eadsv5, Mv2, M, DSv2, Dv2                            | High memory-to-CPU ratio. Great for relational database servers, medium to large caches, and in-memory analytics.                                                                               |
| Storage optimized        | Lsv2, Lsv3, Lasv3                                                                                                                                  | High disk throughput and IO ideal for Big Data, SQL, NoSQL databases, data warehousing and large transactional databases.                                                                       |
| GPU                      | NC, NCv2, NCv3, NCasT4_v3, NC A100 v4, ND, NDv2, NV, NVv3, NVv4, NDasrA100_v4, NDm_A100_v4                                                         | Specialized virtual machines targeted for heavy graphic rendering and video editing, as well as model training and inferencing (ND) with deep learning. Available with single or multiple GPUs. |
| High performance compute | HB, HBv2, HBv3, HBv4, HC, HX                                                                                                                       | Our fastest and most powerful CPU virtual machines with optional high-throughput network interfaces (RDMA).                                                                                     |

### VM storage

- All VM will have at least 2 disks
  - an OS disk: mounted as /dev/svg drive in linux
  - a temporary disk: used to store temporary data. Will be deleted if you redeploy or resize the vm.
  - you can add additional disks (data disk): used to store application data. Nb of data disks you can add depend on the size of the vm

OS disks and data disks are created on Azure blob storage while the temporary disk is directly attached to the host computer

Performance tiers:

- Standard HDD: cheapest for dev and testing
- Standard SSD
- Premium SSD: required for io intensive app
- Ultra SSD

When creating VM, you can choose between managed and unmanaged disks

- In unmanaged disks, you need to take care of the underlying storage account used to store the VHD file.
- In managed disks (recommended option), the underlying storage account is managed by microsoft

### Creating a VM

Creating a VM is straightforward from the portal

### Connecting to a VM

- Public IP:
  - Easiest way to set up. Vm is connected to Network interface which is connected to Network Security Group. NSG should allow inbound connections to port 22. This is the recommending way for testing but not for production
- Jumpbox:
  - Instead of creating a public IP, we create one more subnet: the jumpboxSubnet which contains a VM. We connect to this VM via its public IP and then communicate with the other VMs in the same VNet via their private IP address.
- Azure bastion: platform as a service solution
  - User can connect to the Bastion Host in the AzureBastionSubnet from the Azure Portal. Bastion will then connect to VM without the need to use a public IP

Note: For Linux, recommended way to connect is SSH (TCP/22) with keypairs
For windows, we use RDP (TCP/3389) or WinRM (TCP/5986)

### Configuring high availability: availability sets and zones

- Unplanned hardware maintenance
  - Azure performs a life migration of the VM where the VM is migrated to another physical healthy server, causing the VM to be paused for a short time
- Unexpected downtime
  - Failure of storage components, storage components or other rack related issues
  - Azure will heal the VM by redeploying it to another healthy server in the same datacenter. This can cause a reboot operation and losss of data that is in the temporary drive
- Planned maintenance
  - Necessary to enhance security, and performance of the hardware
  - Most of the time, it does not cause any downtime

In azure, a Geography is composed of multiple regions. Every geography contains at least one region. Within each region there are multiple data centers. These data centers are grouped in availability zones (which are collections of datacenters isolated from other availability zones).

A fault domain is **a set of hardware components that share a single point of failure**. To be fault tolerant to a certain level, you need multiple fault domains at that level. For example, to be rack fault tolerant, your servers and your data must be distributed across multiple rack

Update domains _indicate groups of virtual machines and underlying physical hardware that can be rebooted at the same time_

- When deploying a VM without redundancy, we don't have control to the datacenter or the rack where it is deployed. SLA is 99.9% uptime.

- When using Availability sets, SLA is increased

  - VMs are deployed to multiple fault domains and update domains (the combination is called availability set):
    - resilient to reboot: Virtual machines in the same update domain will be restarted together during planned maintenance. Azure never restarts more than one update domain at a time.
    - resilient to hardware problems: Virtual machines in the same fault domain share a common power source and physical network switch.
  - Availability sets are defined at the datacenter level. If the entire datacenter goes down, the ability set can not save the workload
  - When creating the VM, choose Availability set in the Availability options. Note that for the SLA to increase, you need to have at least 2 VMs in the availability set.
  - Note that if we have multiple machines, it is usually better not to create a public IP and use a load balancer instead

- When using Availability zones, we will deploy the vm to several availability zones so that even if a whole datacenter goes down, we can still access the other availability zones. This increases SLA to 99.99%. Contrary to Availability sets, we don't have to create and availability zone but you need to select it when creating the VM. So you can deploy the VMs to Zone 1 and Zone 2 for ex. Based on the zone selection, azure will create VMs in all availability zones

### VM scale sets

- used to create a group of load balanced VMs and manage them. VM scal sets suupports use of Azure Load Balancer and Application Gateway
- We can increase or decrease the nb of instances based on schedule, metrics or on demand. All VMs in a scale set are created from the same base OS and config.
- VMs in a scale set can be distributed across availability zones for high availability
- For market place and custom images, scale set can scale up to 1000 instances (vs 600 for managed images)

## Load balancing

### Azure load balancer

- Layer 4 load balancer which supports Azure Virtual Machines and Azure Virtual Machines Scale sets as backend
- Two SKUS: Standard and Basic SKU
  - Basic is ideal for testing and dev
  - Standard is ideal for prod because of SLA and HTTPS health probe
- Supports TCP and UDP
- Security managed by NSG
- Public Load balancer:
  - Public load balancer have public IP
  - Incoming traffic IP and port will be mapped to private IP address and port of backend servers
  - Traffic is distributed with load balancing rules
- Internal load balancer
  - does not have public IP
  - Incoming traffic inside the Vnet can be distributed across backend servers
  - Used in internal resources that nees to be accessed from Azure or on premises via VPN connection
- Rules:
  - Load balancing rules: we can create frontend IP to backend IP port mapping
  - Inbound NAT rules: instead of backend pool, we can target a specifig CM and create a NAT rule. Frontend IP and port combination is used to send traffic to IP and port of designated VM.
  - Outbound rule allows backend pool to communicated with the Internet
- Session persistence:
  - None: 5 tuple hash containing Source IP, Source Port, Destination IP, Destination Port and protocol. Requests can be handled by any VM and the chance to get a new VM for every session is very high
  - Client IP: 2 tuple hash (source and dest IP)
  - Client IP and protocal: 3 tuple hash (source and dest IP + protocol)

### Azure Application Gateway

- Layer 7 load balancer
