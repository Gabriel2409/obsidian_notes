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

### Build DNS
cool tutorial: https://github.com/EmilHernvall/dnsguide/blob/b52da3b32b27c81e5c6729ac14fe01fef8b1b593/chapter1.md

DNS specs: https://datatracker.ietf.org/doc/html/rfc1035

DNS packets use UDP and you have at least 3 sections: header, question and answer. (requests do not contain an answer section)
Question and answers can be compressed 

| Section            | Size     | Type              | Purpose                                                                                                |
| ------------------ | -------- | ----------------- | ------------------------------------------------------------------------------------------------------ |
| Header             | 12 Bytes | Header            | Information about the query/response.                                                                  |
| Question Section   | Variable | List of Questions | In practice only a single question indicating the query name (domain) and the record type of interest. |
| Answer Section     | Variable | List of Records   | The relevant records of the requested type.                                                            |
| Authority Section  | Variable | List of Records   | An list of name servers (NS records), used for resolving queries recursively.                          |
| Additional Section | Variable | List of Records   | Additional records, that might be useful. For instance, the corresponding A records for NS records.    |

#### header
| RFC Name | Descriptive Name     | Length             | Description                                                                                                                                                                         |
| -------- | -------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ID       | Packet Identifier    | 16 bits            | A random identifier is assigned to query packets. Response packets must reply with the same id. This is needed to differentiate responses due to the stateless nature of UDP.       |
| QR       | Query Response       | 1 bit              | 0 for queries, 1 for responses.                                                                                                                                                     |
| OPCODE   | Operation Code       | 4 bits             | Typically always 0, see RFC1035 for details.                                                                                                                                        |
| AA       | Authoritative Answer | 1 bit              | Set to 1 if the responding server is authoritative - that is, it "owns" - the domain queried.                                                                                       |
| TC       | Truncated Message    | 1 bit              | Set to 1 if the message length exceeds 512 bytes. Traditionally a hint that the query can be reissued using TCP, for which the length limitation doesn't apply.                     |
| RD       | Recursion Desired    | 1 bit              | Set by the sender of the request if the server should attempt to resolve the query recursively if it does not have an answer readily available.                                     |
| RA       | Recursion Available  | 1 bit              | Set by the server to indicate whether or not recursive queries are allowed.                                                                                                         |
| Z        | Reserved             | 3 bits             | Originally reserved for later use, but now used for DNSSEC queries.                                                                                                                 |
| RCODE    | Response Code        | 4 bits             | Set by the server to indicate the status of the response, i.e. whether or not it was successful or failed, and in the latter case providing details about the cause of the failure. |
| QDCOUNT  | Question Count       | 16 bits            | The number of entries in the Question Section                                                                                                                                       |
| ANCOUNT  | Answer Count         | 16 bits            | The number of entries in the Answer Section                                                                                                                                         |
| NSCOUNT  | Authority Count      | 16 bits            | The number of entries in the Authority Section                                                                                                                                      |
| ARCOUNT  | Additional Count     | 16 bits            | The number of entries in the Additional Section                                                                                                                                     |

#### question
| Field  | Type           | Description                                                          |
| ------ | -------------- | -------------------------------------------------------------------- |
| Name   | Label Sequence | The domain name, encoded as a sequence of labels as described below. |
| Type   | 2-byte Integer | The record type.                                                     |
| Class  | 2-byte Integer | The class, in practice always set to 1.                              |

#### answer

| Field | Type           | Description                                                                       |
| ----- | -------------- | --------------------------------------------------------------------------------- |
| Name  | Label Sequence | The domain name, encoded as a sequence of labels as described below.              |
| Type  | 2-byte Integer | The record type.                                                                  |
| Class | 2-byte Integer | The class, in practice always set to 1.                                           |
| TTL   | 4-byte Integer | Time-To-Live, i.e. how long a record can be cached before it should be requeried. |
| Len   | 2-byte Integer | Length of the record type specific data.                                          |
