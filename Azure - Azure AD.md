---
sr-due: 2023-06-21
sr-interval: 3
sr-ease: 250
---

#azure

https://learn.microsoft.com/en-us/training/modules/configure-azure-active-directory/

## Definition

Azure AD is a managed cloud based identity and directory management service enabling access to Azure services and other SaaS solutions:

- authentication tool: authenticate within Azure AD to have access to all the services
- identity tool: create and manage users and groups
- licensing tool: assign licenses of Microsoft products to users

## Concepts

- **Identity**: object that can be authenticated.
- **Account**: identity that has data associated with it. You can't have an account without an identity.
- **Azure AD account**: identity that's created through Azure AD or another Microsoft cloud service, such as Microsoft 365 (also called a _work or school account_)
- **Azure tenant (directory)**: Single dedicated and trusted instance of Azure AD that represents a single organization. You can have multiple tenants. Go to `Overview_Manage tenants` to see all the tenants. You can switch between tenants by clicking on your profile. Tenants are fully independent.
- **Azure subscription**: used to pay for Azure cloud services. Each subscription is joined to a single tenant. You can have multiple subscriptions.

Note: **A tenant is an Azure Active Directory (Azure AD) entity. It’s the directory in which users, groups, and applications are stored. A subscription is a billing entity that you use to organize access to cloud resources. You can have multiple subscriptions in a single tenant.**

Note: Azure AD is different from traditional Active directory: https://learn.microsoft.com/en-gb/azure/active-directory/fundamentals/compare

## Editions

Azure Active Directory comes in four editions: **Free**, **Microsoft 365 Apps**, **Premium P1**, and **Premium P2**. Premium versions support additionnal features such as self-service password reset

## Follow up

- [[Azure - Hierarchy]]
- [[Azure - Users and groups]]
- [[Azure - Policy]]
- [[Azure - RBAC]]
- [[Azure - Azure AD join]]
- [[Azure - SSPR]]
- [[Azure - Tags]]
- [[Azure - Resource locks]]
