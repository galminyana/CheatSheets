

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

- Maximum 255 characters total, including dots
- Maximum 127 labels, each label separated by a dot
- Each label has maximum 63 characters
- Legal characters are alphanumeric and the dash (-) symbol
- Underscore (_) cannot be used as hostname
- Case insensitive

In the case of `www.example.com` there is 3 labels: `www`, `example`, `com`.

### Resource Records
---

#### **A Record:** Maps a host name to an IP address.
```markup
www.example.com.    3600    IN    A        93.184.216.34
```
#### **AAAA Record:** 
```markup
www.example.com.    3600    IN    AAAA     2607:dead:beef:c607::68
```
#### **PTR Record:** Pointer from an IP address to a hostname
```markup
254.63.231.199.in-addr.arpa.      3600      IN      PTR        ns1.example.com.
```
#### **CNAME (Canonical Name) Record:** Name that points to another CNAME or A record. If a client asks for a CNAME, will have to resolve the refered name.
```markup
www.example.com          60        IN       CNAME       something.cdn.com.
something.cdn.com.       3600      IN       A           10.11.12.13
```
#### **MX (Mail Exchange) Record:** Provides where to send email for the given domain. HAs an extra field, the preference of the mail servers.
```markup
example.org.      7200     IN     MX    10   mx007.postini.com.
example.org.      3600     IN     MX    20   mail1.example.org.
example.org.      3600     IN     MX    20   mail2.example.org.
```
#### **NS (Name Server) Record:** Points to Authoritative Name Servers for the domain.
```markup
example.com.   86400   IN   NS   ns4.example.com.
example.com.   86400   IN   NS   ns2.example.com.
example.com.   86400   IN   NS   ns3.example.com.
example.com.   86400   IN   NS   ns1.example.com.
```
#### **SRV (Service) Record:** Helps to locate services for a domain. For example, an SRV entry for SIP service will look like:
```markup
_sip._tcp.example.com.     300   IN   SRV   0   5   5060   voip1.example.com.
```

  - Where each field for the record name, in order, stands for:
    - Service: the name of the service, SIP in this case on `_sip`.
    - Protocol: the transport protocol of the desired service, TCP or UDP. TCP in this example `_tcp`.
    - Name: the domain name where the service is provided.
  - Where the 4 values after `SRV` are:
    - Priority: lower value means more preferred.
    - Weight: used after priority, higher value means more probable.
    - Port: the TCP or UDP port on which the service is to be found.
    - Target: the canonical hostname of the machine providing the service

#### **TXT Record**: Can store any text to a max of 255 chars.
```markup
example.com.    3600    IN    TXT    "v=spf1 -all"
```
#### **CAA Record:**
#### **TSIG (Transaction Signature) Record:** 
#### **NAPTR (Naming Authority Pointer) Record:** 



