---
sr-due: 2023-06-29
sr-interval: 7
sr-ease: 250
---

#azure

Can be configured at all levels (see [[Azure - Hierarchy]]) by going to `Access Control (IAM)`

## Overview

Enables admins to grant acces to azure resources

- Who: Security Principal: any identity requesting access
- What: Role definition: set of operation a role can perform (written in JSON)
- Where: Scope: limit of access, boundaries

  =>Combination of 3 (role definition atached to a service principal and a scope) = **role assignment** (max 2000 per subscription)

## Azure vs Azure AD roles

TLDR: Azure roles apply to Azure resources. Azure AD roles apply to Azure AD resources such as users, groups, and domains.

Azure RBAC:

- Used to manage access to azure resources
- scope include management groups, subscriptions, resource groups and resources
- example of roles: owner, contributor, reader,...

Azure AD Roles:

- Used to manage azure AD features
- scope is at the tenant level
- example roles: Global Administrator, Billing Administrator

Be sure to follow the principle of least privileges

## Custom roles

https://learn.microsoft.com/en-us/azure/role-based-access-control/role-definitions
Example below

```json
{
	"Name": "Role name",
	"IsCustom": true,
	"Description": "Role description",
	"Actions": [
		"Microsoft.Storage/*/read",
		"Microsoft.Network/*/read",
		"Microsoft.Support/*"
	],
	"NotActions": [],
	"DataActions": [],
	"NotDataActions": [],
	"AssignableScopes": [
		"/subscriptions/{subscriptionId1}",
		"/subscriptions/{subscriptionId2}",
		"/providers/Microsoft.Management/managementGroups/{groupId1}"
	]
}
```
