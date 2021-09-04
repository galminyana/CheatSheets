

### Terminology
---

- **FQDN (Fully Qualified Domain Name):** Complete domain name for a host. Two parts, host name and domain name. For `www.example.com`, host name is `www` and domain `example.com`.
- **Recursive Query:** Initiated by the client stub, asking for a name resolution to DNS server and if it does not know, to check other DNS servers.
- **Iterative Query:** Same as before, but client stubs must be able to follow referrals.
- **Recursive Name Server:** DNS Server receives the query, and:
 - Checks it's cache for the answer.
 - If not in cache, asks other DNS Servers (Root, Autoritative).
- **Authoritative Name Server:** 
- **RR (Resource Record):** Piece of information out of DNS. Examples are A, MX, CNAME...
- **Zone:** Collection of Resource Records for the same domain name suffix.

#### DNS Names Requirements
|---|--|--|
|www.example.com|||
|label|label|label|

1. Maximum 255 characters total, including dots
2. Maximum 127 labels, each label separated by a dot
3. Each label has maximum 63 characters
4. Legal characters are alphanumeric and the dash (-) symbol
5. Underscore (_) cannot be used as hostname
6. Case insensitive

### Resource Records
---

### DNS Message Format
---


