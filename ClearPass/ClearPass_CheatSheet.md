## ClearPass CheatSheet
---

### CLI Commands
---

| Command | Description |
|-|-|
| `cluster list` | Show all cluster members info | 
| `system boot-image` | Show/make active unit images |

## Deal with boot images
```markup
[appadmin@CPPM]# system boot-image
Usage:
    system boot-image [-l] [-a <index>]
Where,
    -l             --  List boot images installed on the system
    -a <index>     --  Set active boot image version. Index is displayed
                       with [-l] option in front of an entry
[appadmin@CPPM]#
```
## Routes
```markup
network ip add <mgmt|data|greN> [-i <id>] <[-s <SrcAddr>] [-d <DestAddr>]> [-g <ViaAddr>]
```
Where:
- i : Id of the rule. Order in the route list
- s <SrcAddr> : Source interface ip address or netmask from where the network ip rule is specified. The allowed values are valid IP Address or Netmask or '0/0'
- d <DestAddr> : Destination interface ip address or netamsk where the network ip rule is specified. The allowed values are -valid IP Address or Netmask or '0/0'
- g <ViaAddr> : Gateway ip address through which the network traffic should flow. The allowed value is valid IP Address
