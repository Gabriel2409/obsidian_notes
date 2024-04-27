---
sr-due: 2024-11-20
sr-interval: 301
sr-ease: 250
reviewed: 2023-07-06
---

#sd

## Definition

A DNS translates a domain name to an IP address

## How it works

### Domain name hierarchy

in www.example.com,

- `.` is the root
- `com` is the TLD (top level domain)
- `example` is the second level domain
- `www` is the subdomain

### Name resolution

- Client machine has usually 1 or 2 DNS (the one given by the company, or the internet box).
- When doing a DNS request, client sends it to the configured DNS (called DNS 1 for the explanation below)
  - If DNS 1 has information in its cache, it returns the IP address
  - If not, it contacts a higher level DNS which will return the DNS server that has authority for the requested domain name (DNS 2). Then DNS 1 will contact DNS 2 to get the IP, cache the IP and return it to the client

Notes:

- Lower level DNS servers cached mappings can become stale due to DNS propagation delay
- DNS results can also be cached by the browser or OS for a certain period of time, determined by the TTL (time to live)

### Components of DNS service

The DNS system needs:

- The DNS protocol which will allow to exchange information about domains
- A DNS server = a software able to handle DNS zones and answer DNS requests

Note: Usually DNS requests are based on UDP protocol on port 53

### DNS records

DNS contain multiple records:

- `A` records map names to **ipv4** addresses: `example.com A 10.11.12.13`
- `AAAA` records map names to **ipv6** addresses: `example.com AAAA 1851:0000:3238:DEF1:0177:0000:0000:0125`
- `MX` records map mail servers
- `CNAME` records are used for aliases so that both map to the same IP: `www.example.com CNAME example.com`
- `PTR` records are used for inverse resolution
- `TEXT` records were originally intended as a place for notes. Now they are used mainly to prevent email spam and verify domain ownership
  - `SPF` records list all the servers that are authorized to send email messages from a domain
  - `DKIM` records work by digitally signing each email using a public-private key pair. This helps verify that the email is actually from the domain it claims to be from. The public key is hosted in a TXT record associated with the domain.
  - `DMARC` records reference the domain's SPF and DKIM policies. It should be stored as `_dmarc.<domain_name>`
  - To prove that we control a domain, we can be asked to edit / add a TXT record
- ...
