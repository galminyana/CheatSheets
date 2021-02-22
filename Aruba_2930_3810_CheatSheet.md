## Aruba CheatSheet (HP Procurve)
---
---

### Various Configs
---
| Command | Description |
|---------|-------------|
| `hostname NAME` | Sets the switch hostname |
| `logging IP` | Sends syslogs to the IP address |
| `logging command` | Enables local command logging |



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
ipv6 access-group "DROP-ALL-V6" out
``` 
