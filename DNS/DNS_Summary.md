# DNS Notes
---
- [Terminology](#terminology)
  * [DNS Names Requirements](#dns-names-requirements)
- [DNS Message Anatomy](#dns-message-anatomy)
- [DNS Header](#dns-header)
  * [DNS Questions](#dns-questions)
  * [DNS Answers](#dns-answers)
  * [Header Examples](#header-examples)
    + [Response Codes](#response-codes)
    + [Header Flags](#header-flags)
- [Resource Records Format](#resource-records-format)
- [Common Resource Records](#common-resource-records)
  * [SOA, Start of Authority](#soa--start-of-authority)
  * [A Record](#a-record)
  * [AAAA Record](#aaaa-record)
  * [PTR Record](#ptr-record)
  * [CNAME (Canonical Name) Record](#cname--canonical-name--record)
  * [MX (Mail Exchange) Record](#mx--mail-exchange--record)
  * [NS (Name Server) Record](#ns--name-server--record)
  * [SRV (Service) Record](#srv--service--record)
  * [TXT Record](#txt-record)
  * [HINFO:](#hinfo-)
- [Name Resolution Process](#name-resolution-process)
- [Root Servers](#root-servers)

### Terminology
---

- **FQDN (Fully Qualified Domain Name):** Complete domain name for a host. Two parts, host name and domain name. For `www.example.com`, host name is `www` and domain `example.com`.
- **Recursive Query:** Initiated by the client stub, asking for a name resolution to DNS server and if it does not know, to check other DNS servers.
- **Iterative Query:** Same as before, but client stubs must be able to follow referrals.
- **Recursive Name Server:** A recursive nameserver is one that answers queries by asking other nameservers for the answer. DNS Server receives the query, and:
  - Checks it's cache for the answer.
  - If not in cache, asks other DNS Servers (Root, Autoritative).
- **Authoritative Name Server:** Provides answers to DNS queries. It does not provides just cached answers that were obtained from another name server, it only returns answers to queries about domain names that are installed in its configuration system.
- **Caching Name Server:** Store DNS query results for a period of time determined in the config­uration (TTL) of each domain name record. Caching name servers are often also recursive name servers, as they perform every step necessary to answer any DNS query they receive.
- **RR (Resource Record):** Piece of information out of DNS. Examples are A, MX, CNAME...
- **Zone:** Collection of Resource Records for the same domain name suffix.

#### DNS Names Requirements

- Maximum 255 characters total, including dots
- Maximum 127 labels, each label separated by a dot
- Each label has maximum 63 characters
- Legal characters are alphanumeric and the dash (-) symbol
- Underscore (`_`) cannot be used as hostname
- Case insensitive

In the case of `www.example.com` there are 3 labels: `www`, `example`, `com`.

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

### DNS Header
---
```markup
                                1  1  1  1  1  1
  0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                    ID                         |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|QR|   Opcode  |AA|TC|RD|RA|     Z  |   RCODE   |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                   QDCOUNT                     |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                   ANCOUNT                     |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                   NSCOUNT                     |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                   ARCOUNT                     |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```
- **ID**: A 16 bit identifier assigned by the program that generates any kind of query. This identifier is copied the corresponding reply and can be used by the requester to match up replies to outstanding queries. 
- **QR**: A one bit field that specifies whether this message is a query (0), or a response (1).
- **OPCODE**: A four bit field that specifies kind of query in this message. 
- **AA**: Authoritative Answer. This bit is only meaningful in responses, and specifies that the responding name server is an authority for the domain name in question section.
- **TC**: Truncation.Specifies that this message was truncated. 
- **RD**: Recursion Desired. This bit directs the name server to pursue the query recursively. 
- **RA**: Recursion Available. This bit is set or cleared in a response, and denotes whether recursive query support is available in the name server. Recursive query support is optional. 
- **Z**: Reserved for future use. 
- **RCODE**: Response code. This 4 bit field is set as part of responses. The values have the following interpretation:
  - **0**: No error condition
  - **1**: Format error - The name server was unable to interpret the query.
  - **2**: Server failure - The name server was unable to process this query due to a problem with the name server.
  - **3**: Name Error - Meaningful only for responses from an authoritative name server, this code signifies that the domain name referenced in the query does not exist.
  - **4**: Not Implemented - The name server does not support the requested kind of query.
  - **5**: Refused - The name server refuses to perform the specified operation for policy reasons
- **QDCOUNT**: Unsigned 16 bit integer specifying the number of entries in the question section. 
- **ANCOUNT**: Unsigned 16 bit integer specifying the number of resource records in the answer section. You should set this field to 0, indicating you are not providing any answers.
- **NSCOUNT**: Unsigned 16 bit integer specifying the number of name server resource records in the authority records section. 
- **ARCOUNT**: Unsigned 16 bit integer specifying the number of resource records in the additional records se

#### DNS Questions
---
```markup
                                1  1  1  1  1  1
  0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                     QNAME                     |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                     QTYPE                     |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                     QCLASS                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```
- **QNAME**: A domain name represented as a sequence of labels, where each label consists of a lengt octet followed by that number of octets. The domain name terminates with the zero length octet for the null label of the root. 
- **QTYPE**: A two octet code which specifies the type of the query. 
- **QCLASS**: A two octet code that specifies the class of the query. 
#### DNS Answers
---
```markup
                                1  1  1  1  1  1
  0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                 NAME                          |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--|
|                 TYPE                          |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                 CLASS                         |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                 TTL                           |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                 RDLENGTH                      |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                 RDATA                         |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```
- **NAME**: The domain name that was queried, in the same format as the QNAME in the questions.
- **TYPE**: Two octets containing one of th type codes. This field specifies the meaning of the data in the RDATA field. 
- **CLASS**: Two octets which specify the class of the data in the RDATA field. 
- **TTL**: The number of seconds the results can be cached.
- **RDLENGTH**: The length of the RDATA field.
- **RDATA**: The data of the response. The format is dependent on the TYPE field: 
  - If the TYPE is 0x0001 for A records, then this is the IP address (4 octets). 
  - If the type is 0x0005 for CNAMEs, then this is the name of the alias. 
  - If the type is 0x0002 for name servers, then this is the name of the server. 
  - If the type is 0x000f for mail servers, the format is.

```markup
+--+--+--+---+
| PREFERENCE |
+--+--+--+---+
| EXCHANGE   |
+--+--+--+---+
```
#### Header Examples
---
In `dig` responses, we can see the header, with the most representative fields like `id`, `status` and `flags`.
```markup
;; ->>HEADER<<-opcode: QUERY, status: NOERROR, id: 40105
;; flags: qr aa, QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL:
```
##### Response Codes
---
|Code |Name |Description|
|-|-|-|
|0| NOERROR| No error|
|1| FORMERR| Format error, usually indicates client sending malformed query|
|2| SERVFAIL| Server failure, generic error message indicating error occured on server side|
|3| NXDOMAIN| Non-existent domain, indicates the domain name queried does not exist|
|5| REFUSED| Server configuration does not allow it to respond to client|

##### Header Flags
---
|Flag|Name|"0"|"1"|
|-|-|-|-|
|qr|Query Response|mMssage is a Question|Message is an Answer|
|aa|Authoritative Answer|Not Authoritative|Authoritative|
|tc|Trucation|Message Not Truncated|Message Truncated, retry on TPC|
|rd|Recursion Desired|Iterative Query|Recursive Query|
|ra|Recursion Available|Not Available|Available|
|ad|Authenticated Data|Not Auth|Authenticated|

### Resource Records Format
---
|NAME  |TTL  |TYPE |RDATA|
|------|-----|-----|-----|
|Domain name to witch this RR pertains|32bit int for the time a RR should be cached|Specifies what data is in RDATA|A variable lenght string to describe the resource|



### Common Resource Records
---

Check Wikipedia for a full list of all existing records, including NULL ones: [https://en.wikipedia.org/wiki/List_of_DNS_record_types](https://en.wikipedia.org/wiki/List_of_DNS_record_types)
#### SOA, Start of Authority
Indicates the basic properties of the domain name server and the zone that the domain is in. Each zone file can contain only one SOA record
- REFRESH: how often a secondary should contact primary for update
- RETRY: how often a secondary should try contacting primary should it fail to reach primary the previous attempt
- EXPIRE: how long a secondary can hold on to the zone data when it cannot reach primary
- MINIMUM: how long recursive name servers can cache a negative answer such as

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

### Name Resolution Process
---
More information here: [http://www.tcpipguide.com/free/t_DNSNameResolutionProcess-2.htm](http://www.tcpipguide.com/free/t_DNSNameResolutionProcess-2.htm)
Steps for resolving `www.example.com`would be:
1) Recursive Query: Stub resolver, afer not having cached the information, reaches to a recursive resolver, according to local configuration. Sends a recursive query (RD flag to 1)
2) Iterative Query: Recursive resolver can have the answer in the cache, then responds with it to the stub. IF not cached, asks to other name servers, defaults to root name servers
3) Referral: Root servers responds with a referral, pointing the recursive resolver with the name servers where to ask. Root servers are authoritative for the first level domain, the dot (.).
4) Iterative Query: Resolver keeps the lists of name servers obtained from previous step. Then chooses a server from the list, and sends an iterative query for the domain.
5) Referral: Delegated domain responds with another referral to the next level of the FQDN. As it's an authoritative server for the TLD domain (Second level)
6) Iterative Query: Recursive resolvers caches answers from previous step, and asks to one of the servers. Those servers, will be authooritative servers for the 'example.com` domain.
7) Authoritative Answer: The authoritative name server sends the authoritative answer to the recursive resolver (AA field to "1").
8) Non-Authoritative Answer: Recursive resolvers caches the answer, and sends it to the stub resolver, but with the AA field changed to "0" (non-authoritative).

### Root Servers
---

|Letter|Name|IPv4|IPv6|Operated By|
|-|-|-|-|-|
|A| a.root-servers.net| 198.41.0.4| 2001:503:ba3e::2:30 |Verisign|
|B| b.root-servers.net| 192.228.79.201 |2001:478:65::53| University of Southern California - ISi|
|C| c.root-servers.net| 192.33.4.12| 2001:500:2::c| Cognet Communications|
|D| d.root-servers.net| 199.7.91.13| 2001:500:2d::d| University of Maryland|
|E| e.root-servers.net| 192.203.230.10| N/A |NASA|
|F| f.root-servers.net| 192.5.5.241| 2001:500:21::f| Internet Systems Consortium|
|G| g.root-servers.net| 192.112.36.4| N/A| Defense Information Systems Agency|
|H| h.root-servers.net| 128.63.2.53| 2001:500:1:8031:235| U.S. Army Research Lab|
|I| i.root-servers.net| 192.36.148.17| 2001:7fe::53 |Netnod|
|J| j.root-servers.net| 192.58.128.30| 2001:503:c27::2:30 |Verisign|
|K| k.root-servers.net| 193.0.14.129| 2001:7fd::1| RIPE NCC|
|L| I.root-servers.net| 199.7.83.42| 2001:500:3::42| ICANN|
|M| m.root-servers.net| 202.12.27.33| 2001:dc3::35| WIDE Project|

