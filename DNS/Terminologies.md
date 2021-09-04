

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

1. Maximum 255 characters total, including dots
2. Maximum 127 labels, each label separated by a dot
3. Each label has maximum 63 characters
4. Legal characters are alphanumeric and the dash (-) symbol
5. Underscore (_) cannot be used as hostname
6. Case insensitive

In the case of `www.example.com` there is 3 labels: `www`, `example`, `com`.

### Resource Records
---

- **A Record:** Maps a host name to an IP address.
```markup
www.example.com.    3600    IN    A        93.184.216.34
```
- **AAAA Record:** 
```markup
www.example.com.    3600    IN    AAAA     2607:dead:beef:c607::68
```
- **PTR Record:** Pointer from an IP address to a hostname
```markup
254.63.231.199.in-addr.arpa.      3600      IN      PTR        ns1.example.com.
```
- **CNAME (Canonical Name) Record:** Name that points to another CNAME or A record. If a client asks for a CNAME, will have to resolve the refered name.
```markup
www.example.com          60        IN       CNAME       something.cdn.com.
something.cdn.com.       3600      IN       A           10.11.12.13
```
- **MX (Mail Exchange) Record:** Provides where to send email for the given domain. HAs an extra field, the preference of the mail servers.
```markup
example.org.      7200     IN     MX    10   mx007.postini.com.
example.org.      3600     IN     MX    20   mail1.example.org.
example.org.      3600     IN     MX    20   mail2.example.org.
```
- **NS (Name Server) Record:** Points to Authoritative Name Servers for the domain.
```markup
example.com.   86400   IN   NS   ns4.example.com.
example.com.   86400   IN   NS   ns2.example.com.
example.com.   86400   IN   NS   ns3.example.com.
example.com.   86400   IN   NS   ns1.example.com.
```
- **SRV (Service) Record:** 

### DNS Message Format
---


