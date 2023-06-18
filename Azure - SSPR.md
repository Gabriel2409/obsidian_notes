#azure
https://learn.microsoft.com/en-gb/training/modules/allow-users-reset-their-password/2-self-service-password-reset

Click on `Manage>Users>Password reset` (premium feature)

Set up multiple methods to reset password without having to go through IT help desk

Azure supports six different ways to authenticate reset requests:

- Mobile app notification (with microsoft authenticator)
- Mobile app code: also with authenticator but you enter a code from the app
- email: sends email to another email external to microsoft
- mobile phone: needs a phone number
- office phone: nonmobile phone number
- security questions

You can specify the min number of methods to enable

Note:

- A strong, two-method authentication policy is always applied to accounts with an administrator role, regardless of your configuration for other users.
- The security questions method isn't available to accounts that are associated with an administrator role.
