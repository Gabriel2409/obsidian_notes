#azure

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

see lab https://learn.microsoft.com/en-us/training/modules/configure-user-group-accounts/7-simulation-user-groups
