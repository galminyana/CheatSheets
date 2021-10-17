

### Terminology
---

- **FQDN (Fully Qualified Domain Name):** Complete domain name for a host. Two parts, host name and domain name. For `www.example.com`, host name is `www` and domain `example.com`.
- **Recursive Query:** Initiated by the client stub, asking for a name resolution to DNS server and if it does not know, to check other DNS servers.
- **Iterative Query:** Same as before, but client stubs must be able to follow referrals.
- **Recursive Name Server:** A recursive nameserver is one that answers queries by asking other nameservers for the answer. DNS Server receives the query, and:
  - Checks it's cache for the answer.
  - If not in cache, asks other DNS Servers (Root, Autoritative).
- **Authoritative Name Server:** Provides answers to DNS queries. It does not provides just cached answers that were obtained from another name server, it only returns answers to queries about domain names that are installed in its config­uration system.
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

### DNS Message Anatomy
---
```markup

                        C:\dig>dig example.com A

                        ; <<>> DiG 9.16.21 <<>> example.com A
                        ;; global options: +cmd
+--------------+        ;; Got answer:
|    Header    |        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55320
+--------------+        ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 1
|    Question  |        ;; OPT PSEUDOSECTION:
+--------------+        ; EDNS: version: 0, flags:; udp: 1460
|    Answer    |        ;; QUESTION SECTION:
+--------------+        ;example.com.                   IN      A
|   Authority  |
+--------------+        ;; ANSWER SECTION:
|  Additional  |        example.com.            77941   IN      A       93.184.216.34
+--------------+
                         ;; AUTHORITY SECTION:
                        example.com.            77941   IN      NS      a.iana-servers.net.
                        example.com.            77941   IN      NS      b.iana-servers.net.

                        ;; Query time: 46 msec
                        ;; SERVER: 80.58.61.250#53(80.58.61.250)
                        ;; WHEN: Sun Oct 17 10:46:11 Hora de verano romance 2021
                        ;; MSG SIZE  rcvd: 104
```
### Resource Records Format
---
|NAME  |TTL  |TYPE |RDATA|
|------|-----|-----|-----|
|Domain name to witch this RR pertains|32bit int for the time a RR should be cached|Specifies what data is in RDATA|A variable lenght string to describe the resource|

### Common Resource Records
---
#### SOA, Start of Authority
Indicates the basic properties of the domain name server and the zone that the domain is in. Each zone file can contain only one SOA record
- Serial is the serial number of the zone
```markup
google.es.         60      IN      SOA     ns1.google.com. dns-admin.google.com. 
                                           403574256   ;Serial
                                           900         ;Refresh
                                           900         ;Retry
                                           1800        ;Expire
                                           60          ;Minimum
```
#### A Record
Maps a host name to an IP address.
```markup
www.example.com.    3600    IN    A        93.184.216.34
```
#### AAAA Record 
```markup
www.example.com.    3600    IN    AAAA     2607:dead:beef:c607::68
```
#### PTR Record 
Pointer from an IP address to a hostname
```markup
254.63.231.199.in-addr.arpa.      3600      IN      PTR        ns1.example.com.
```
#### CNAME (Canonical Name) Record
Name that points to another CNAME or A record. If a client asks for a CNAME, will have to resolve the refered name.
```markup
www.example.com          60        IN       CNAME       something.cdn.com.
something.cdn.com.       3600      IN       A           10.11.12.13
```
#### MX (Mail Exchange) Record
Provides where to send email for the given domain. HAs an extra field, the preference of the mail servers.
```markup
example.org.      7200     IN     MX    10   mx007.postini.com.
example.org.      3600     IN     MX    20   mail1.example.org.
example.org.      3600     IN     MX    20   mail2.example.org.
```
#### NS (Name Server) Record
Points to Authoritative Name Servers for the domain.
```markup
example.com.   86400   IN   NS   ns4.example.com.
example.com.   86400   IN   NS   ns2.example.com.
example.com.   86400   IN   NS   ns3.example.com.
example.com.   86400   IN   NS   ns1.example.com.
```
#### SRV (Service) Record
Helps to locate services for a domain. For example, an SRV entry for SIP service will look like:
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

#### TXT Record
Can store any text to a max of 255 chars.
```markup
example.com.    3600    IN    TXT    "v=spf1 -all"
```
#### HINFO: 
Host information about the CPU type and operating system of subject of the query.
```markup

```
#### CAA Record
#### TSIG (Transaction Signature) Record 
#### NAPTR (Naming Authority Pointer) Record 



