## Aruba CheatSheet (HP Procurve)
---
  * [Various Configs](#various-configs)
    + [Time](#time)
    + [SNMPv3](#snmpv3)
  * [Firmware](#firmware)
    + [From USB](#from-usb)
  * [Information](#information)
  * [Passwords Security](#passwords-security)
  * [Console Timeouts](#console-timeouts)
  * [DHCP- Snooping](#dhcp--snooping)
  * [Loop-Protect](#loop-protect)
  * [Client Tracker](#client-tracker)
  * [Drop IPv6 Traffic](#drop-ipv6-traffic)
  
### Various Configs
---
| Command | Description |
|---------|-------------|
| `hostname NAME` | Sets the switch hostname |
| `logging IP` | Sends syslogs to the IP address |
| `logging command` | Enables local command logging |

#### Time
```markup
timesync sntp
sntp unicast
sntp server priority 1 IP
time daylight-time-rule western-europe
time timezone 60
```
#### SNMPv3
```markup
snmpv3 enable 
snmpv3 user SNMP_USER auth sha PASSWD priv aes PASSWD
snmpv3 group operatorauth user "SNMP_USER" sec-model ver3
no snmpv3 user initial
snmpv3 only
no snmp-server community public
snmp-server response-source dst-ip-of-request
snmp-server contact "EMAIL" location "LOCATION" 
ip authorized-managers 10.1.0.0 255.255.0.0 access manager
```
### Firmware
---
#### From USB 
```markup
copy usb flash WC_16_08_0001.swi primary
boot system flash primary
```

### Information
---

| Command | Description |
|---------|-------------|
| `chassislocate` | Turns on/off locator led |
| `chassislocate vsf member ID <on/off>` | Turns locator led for the stack member |
| `sh version` | Shows software version |

### Passwords Security
---
| Command | Description |
|---------|-------------|
| `include-credentials` | Includes the credentials on the config file |
| `encrypt-credentials` | Encrypts the included credentials |
| `password non-plaintext-sha2`| Credentils are encrypted using SHA256 |

### Console Timeouts
---
| Command | Description |
|---------|-------------|
| `console idle-timeout SECS` | Disconects iddle session after SECS seconds |
| `console idle-timeout serial-usb SECS` | |

### DHCP- Snooping
---
| Command | Description |
|---------|-------------|
| `dhcp-snooping` | Enables DHCP Snooping |
| `dhcp-snooping vlan VLAN` | Applies DHCP snooping on VLAN |
| `dhcp-snooping trust PORT`| Trusts all DHCP from the PORT |
| `sh dhcp-snooping stats` | Shows dhcp snooping statistics |
| `sh dhcp-snooping binding` | Shows dhcp snooping bindings information |
| `sh dhcp-snooping server-details` | DHCP snooping server details |

### Loop-Protect
---
| Command | Description |
|---------|-------------|
| `loop-protect PORTS` | Enables loop protection in the especified ports|
| `loop-protect PORTS receiver-action send-recv-dis` | |
| `loop-protect disable-timer TIME` | Disables ports for TIME seconds when a loop detected |
| `loop-protect trap loop-detected` | Sends a Trap when loop detected |
| `sh loop-protect [PORT]` | Shows loop protction info [for the port] |

### Client Tracker
---
| Command | Description |
|---------|-------------|
| `ip client-tracker`| |
| `ip client-tracker probe-delay SEC` | Delays ARP probes by SEC seconds | 

### Drop IPv6 Traffic
---
1- Create access list to DROP IPv6:
```markup
ipv6 access-list "DROP-ALL-V6"
  10 deny ipv6 ::/0 ::/0
 exit
 ```
 2- Apply ACL to interfaces
 ```markup
ipv6 access-group "DROP-ALL-V6" in
``` 
### Port Mirroring
---
#### Local Mirroring
Can mirror per switch port, VLAN, or mac addresses.
1- Create he mirror and assign the local mirroring port:
```markup
(Switch)# mirror SESSION port PORT [name NAME]
```
 - SESSION: ID for the session, value 1-4
 - PORT: Switch port to send mirrored traffic

2- Assign the monitored ports, vlans or mac addresses to the created mirroring session. This will send the traffic to the PORT assigned in previous step:
 - By port: The traffic from a switch port sent to the mirror session
```markup
(Switch)# interface {port | trunk | mesh} monitor all {in | out | both} mirror {session-# | name-str} [{session-# | name-str}] [{session-# | name-str}] | [{session-# | name-str}] [no-tag-added]
```
 - By VLAN: Traffic from a VLAN sent to the mirror session
```markup
(switch) vlan vid-# monitor all {in | out | both} mirror {session-# | name-str} [{session-# | name-str}] [{session-# | name-str}] [{session-# | name-str}]
```
 - By mac-address: All traffic from the mac address sent to the session
```markup
(switch)# monitor mac mac-addr [src | dest | both] mirror {session-# | name-str} [{session-# | name-str}] [{session-# | name-str}] [{session-# | name-str}]
```
#### Remote Mirroring
