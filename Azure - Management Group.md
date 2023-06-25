---
sr-due: 2023-07-01
sr-interval: 9
sr-ease: 250
---

#azure

see [[Azure - Hierarchy]]

- Management groups are the scope above [[Azure - Subscriptions]]
- They can be nested up to 6 levels
- By default, all new subscriptions are placed under the top-level management group, or _root group_.
- All subscriptions within a management group automatically inherit the conditions applied to that management group.

Note: to be able to access root management group, you need to go to azure AD, then properties and select Yes in `Access management for Azure resources`
