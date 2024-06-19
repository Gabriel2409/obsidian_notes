---
id: JWT Token
aliases: []
tags: []
---
## What is a JWT Token

A JSON Web Token (JWT) is a compact, URL-safe means of representing claims to be transferred between two parties. The token is composed of three parts: the header, the payload, and the signature

### Header

The header typically consists of two parts: the type of the token (which is JWT) and the signing algorithm being used, such as HMAC SHA256 or RSA.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload

The payload contains the claims. Claims are statements about an entity (typically, the user) and additional data

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

### Signature

To create the signature part, you have to take the encoded header, the encoded payload, a secret, and the algorithm specified in the header, and sign that. For example, if you want to use the HMAC SHA256 algorithm, the signature will be created in the following way:

```bash
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

Note: JWTs can be signed using a secret (with the HMAC algorithm) or a public/private key pair using RSA or ECDSA.

Once the signature is computed, you actually send
`header.payload.signature` (all parts in b64 separated by a .)

Then the client will decode `header.payload` with the secret and check that it can correctly recompute the signature

## Example of JWT

With [[OAuth and OIDC (openid)| OpenId connect]], You can retrieve tokens that look like this. This is not the idToken sent by google.

```json
{
  "header": {
    "alg": "RS256",
    "kid": "<public-key-id>",
    "typ": "JWT"
  },
  "payload": {
    "name": "John Doe",
    "picture": "<link-to-profile-picture>",
    "iss": "https://securetoken.google.com/<your-gcp-app>",
    "aud": "<your-gcp-app>",
    "auth_time": <authentication-timestamp>,
    "user_id": "<firebase-user-id>",
    "sub": "<firebase-user-id>",
    "iat": <jwt-issued-at-timestamp>,
    "exp": <jwt-expiration-timestamp>,
    "email": "<user-email>",
    "email_verified": <boolean>,
    "firebase": {
      "identities": {
        "google.com": [
          "<google-user-unique-id>"
        ]
      },
      "sign_in_provider": "google.com"
    }
  }
}
```

