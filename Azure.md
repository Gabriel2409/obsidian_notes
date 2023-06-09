## Azure AD

Go to `>Azure Active Directory`

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