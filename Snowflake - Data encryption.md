#snowflake


## Data encryption

- Encryption at rest
  - All data that snowflake has control over is AES-256 encrypted (table data, internal storage data, virtual warehouse and query results cache,...)
- Encryption in transit:
  - Secure https is used with TLS 1.2 protocol

=> End to End encryption

- When uploading to an internal stage, snowflake transparently encrypts data then encrypts with a different key to internal storage
- When uploading to an external stage, client is responsible for encryption and must provide snowflake with information to decrypt the data

Snowflake uses a hierarchical key model where each key encrypts the key below it and the final key encrypts the user's data:

- Root key
- Account master keys: encrypts an account
- Table master keys: encrypts a database
- File keys: each datafile or micropartition is encrypted with a separate key

This limits the amount of data a key protects.

- Snowflake also uses key rotation = practise of transparently replacing existing account and table encryption keys every 30 days with a new key. Previous key is only used to decrypt data.
- If we enable rekeying, after 1 year, key is retired and associated information is reencrypted with newest key: `ALTER ACCOUNT SET PERIODIC_DATA_REKEYING = TRUE;`

- Tri-secret secure allows to encrypt snowflake with a composite master key: one part customer managed key and one part account managed key by snowflake