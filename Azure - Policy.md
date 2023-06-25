---
sr-due: 2023-07-03
sr-interval: 11
sr-ease: 250
---

#azure

## Definition

Policies can be used to define organizational standards and identify non-compliant resources. They can be defined anywhere in the [[Azure - Hierarchy|hierarchy]]

- Definition: JSON Document to describe the effect
- Scope: Where we enforce the policy
- Assignment: assign a policy to a scope
- Compliance: evaluate the compliance and understand non compliant resources

Example of policies:

- Allowed resources types
- Allowed virtual machines
- Allowed locations
- Enforce tags that need to be added
- Inherit tags from subscription or resource group (by default it is not the case)
- Allowed resource group locations

Initiatives: group policies together

Policies can be enforced or disabled (if they are disabled, we still see the non compliant resources)

Azure Policy can be used to automatically remediate non-compliant resources by triggering remediation tasks when a policy violation occurs, helping to maintain compliance and enforce governance in real-time. To do so you need to either create a user managed identity or a system managed identity and assign the correct permissions

## Steps

- Create policy definition. Example: prevent VMs in your organization from being deployed, if they're exposed to a public IP address.
- Create initiative definition by grouping together policies to make sure resources are compliant
- Scope initiative definition: initiative can be assigned to a management group, a subscription, etc..
- Determine compliance for matched resources

Example lab: https://learn.microsoft.com/en-us/training/modules/configure-azure-policy/9-simulation-policy
