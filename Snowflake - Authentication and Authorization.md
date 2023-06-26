#snowflake

## Authentication and Authorization

### User authentication

- default method of authentication
- When a SYSADMIN creates a user, he can specify a very short password that does not meet the security requirements. In this case, it is better to force a change on first login

```sql
CREATE USER USER1
PASSWORD ='ABC123'
DEFAULT_ROLE = MYROLE
MUST_CHANGE_PASSWORD = TRUE;
```

### Multi factor authentication

- In snowflake, powered by Duo Security service, managed by snowflake
- user must download the DUO app
- MFA is enabled on a per-user basis and only via the UI
- Snowflake recommends that ACCOUNTADMIN have MFA

```sql
-- specifies the nb of minutes to temporarily disable MFA for the user to log in
ALTER USER USER1 SET MINS_TO_BYPASS_MFA=10;

-- disables MFA for the user, cancelling their enrolment
ALTER USER USER1 SET DISABLE_MFA=TRUE;

-- MFA token caching reduces the nb of prompts that must be acknowledged
ALTER USER USER1 SET ALLOWS_CLIENT_MFA_CACHING=TRUE;
```

### Federated Authentication

- enables users to connect to snowflake using secure SSO
- Snowflake can delegate authentication responsibility to an SAML 2.0 compliant external identity provaider (IdP) with native support for Okta and ADFS IdPs
- In a federated environment, snowflake is referred to as a Service Provider (SP)
- This is done by creating an object called security integration

```sql
CREATE SECURITY INTEGRATION <name>
	TYPE = SAML2
	ENABLED = TRUE
	SAML2_ISSUER = 'XXX'
	SAML2_SSO_URL = 'XXX'
	SAML2_PROVIDER = 'XXX'
	SAML2_X509_CERT = 'XXX'
```

### Key Pair authentication

- alternative to username/password authentication when connecting via a client (not the UI)
- generate public private key with openssl
- Then in snowflake:

```sql
ALTER USER USER1 SET RSA_PUBLIC_KEY='xxxx';
```

- Configure key roation

```sql
ALTER USER USER1 SET RSA_PUBLIC_KEY_2='yyyy';
ALTER USER USER1 UNSET RSA_PUBLIC_KEY='xxxx';
```

### OAuth and SCIM

- Snowflake supports the OAuth2 protocol with 2 pathways

  - Snowflake OAuth
  - External OAuth

- SCIM: System for Cross-domain Identity Management is used to manage snowflake users and roles using RESTful APIs.
  - for ex, an IdP like ADFS uses a SCIM client to make RESTful APIs requests to the Snowflake SCIM server
  - SCIM can be used in the user lifecycle (creating, updating settings,...)