#sd #todo

## Establishing connexion

1. **Server Setup:**

   - The server generates a private key and uses it to create a Certificate Signing Request (CSR). The CSR includes the server's public key and information about the server (e.g., domain name).

2. **CA Validation:**

   - The server submits the CSR to a Certificate Authority (CA) for validation. The CA verifies the information in the CSR and issues a digital certificate, signed with the CA's private key.

3. **Certificate Installation:**

   - The server installs the issued digital certificate along with its private key.

4. **Client Access:**

   - When a client accesses the server over HTTPS, the server presents its digital certificate during the TLS/SSL handshake.

5. **Client Certificate Validation:**

   - The client validates the server's certificate. This involves checking the digital signature on the certificate using the CA's public key, verifying the certificate's expiration date, and ensuring it is not revoked.

6. **Key Exchange:**

   - The client generates a symmetric key (session key) for encryption purposes. This is not directly related to the server's public key but is part of the key exchange process.

7. **Public Key Encryption:**

   - The client encrypts the symmetric key with the server's public key from the presented certificate. This encrypted symmetric key is sent back to the server.

8. **Symmetric Key Usage:**
   - Both the client and server now possess the same symmetric key. This key is used for the remainder of the session to encrypt and decrypt data, providing a faster and more efficient method of secure communication compared to asymmetric encryption.

## Validating the CA

The client can trust the Certificate Authority (CA) through a process known as a "chain of trust.

1. **Built-in Root Certificates:**

   - Web browsers and operating systems come pre-installed with a set of root certificates from well-known and trusted CAs. These root certificates act as the foundation of trust.

2. **CA Signature:**

   - When a server presents its certificate during the TLS handshake, the client can verify the authenticity of the certificate by checking the digital signature. This signature is created using the private key of the CA.

3. **Root Certificate Verification:**

   - The client uses its pre-installed root certificates to verify the signature on the server's certificate. If the signature is valid and matches a trusted root certificate, the client considers the server's certificate valid.

4. **Chain of Trust:**

   - Most CAs do not directly sign the server's certificate with their root key due to security reasons. Instead, they use an intermediate certificate, also signed by the root. This creates a chain of trust: the server's certificate is signed by an intermediate CA, and the intermediate CA's certificate is signed by the root CA.

5. **Intermediate Certificate:**

   - The server provides its certificate and any necessary intermediate certificates during the TLS handshake. The client checks the entire chain, ensuring that each certificate in the chain is signed by the one preceding it, up to a trusted root certificate.

6. **Validity Checks:**

   - The client also checks that each certificate in the chain is not expired and has not been revoked. CAs maintain Certificate Revocation Lists (CRLs) or use the Online Certificate Status Protocol (OCSP) for this purpose.

7. **Browser/OS Updates:**
   - Trust in CAs is further maintained through periodic updates to root certificates by browser and operating system vendors. This allows them to remove or update certificates in case of security incidents or changes in trustworthiness.
