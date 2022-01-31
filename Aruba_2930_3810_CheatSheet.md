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
  * [Port Mirroring](#port-mirroring)
    + [Local Mirror](#local-mirroring)
    + [Remote Mirror](#remote-mirroring)
  
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

- Create the mirror and assign the local mirroring port:

```markup
(Switch)# mirror SESSION_ID port PORT [name NAME]
```

Where SESSION_ID stands for ID for the session, value 1-4, and PORT stands for Switch port to send mirrored traffic

- Assign the monitored ports, vlans or mac addresses to the created mirroring session. This will send the traffic to the PORT assigned in previous step:

  - By port: The traffic from a switch port sent to the mirror session
  - By VLAN: Traffic from a VLAN sent to the mirror session
  - By mac-address: All traffic from the mac address sent to the session

```markup
(Switch)# interface {port | trunk | mesh} monitor all {in | out | both} mirror {session-# | name-str} [{session-# | name-str}] [{session-# | name-str}] | [{session-# | name-str}] [no-tag-added]

(switch) vlan vid-# monitor all {in | out | both} mirror {session-# | name-str} [{session-# | name-str}] [{session-# | name-str}] [{session-# | name-str}]

(switch)# monitor mac mac-addr [src | dest | both] mirror {session-# | name-str} [{session-# | name-str}] [{session-# | name-str}] [{session-# | name-str}]
```
#### Remote Mirroring

- Source Switch
  - Create the session, assign the source ip address and source udp port used in the source switch and assign the destination ip address of the remote switch
  - Assign the monitored ports, vlans or mac addresses to any of the created remote port mirroring sessions
  
```markup
mirror SESSION_ID [name name-str] remote ip src-ip src-udp-port dst-ip [truncation]

interface {port | trunk | mesh} monitor all {in | out | both} mirror {session-# | name-str} [{session-# | name-str}] [{session-# | name-str}] | [{session-# | name-str}] [no-tag-added]

vlan vid-# monitor all {in | out | both} mirror {session-# | name-str} [{session-# | name-str}] [{session-# | name-str}] [{session-# | name-str}]

monitor mac mac-addr [src | dest | both] mirror {session-# | name-str} [{session-# | name-str}] [{session-# | name-str}] [{session-# | name-str}]
```

- Destination Switch
  - Use the same parameters (source ip address, source udp port, destination ip address) employed in the source switch configuration, and assign the mirroring port:

```markup
mirror endpoint ip src-ip src-udp-port dst-ip port exit-port-#
```
### IGMP
---
#### Enable / Disable IGMP
```markup
(config) [no] vlan VLAN_ID ip igmp
```
By default, enables IGMP Querier
#### Enable / Disable Querier
```markup
[no] vlan <vid> ip igmp querier
```
#### Per Port IGMP traffic filters
```markup
vlan <vid> ip igmp [ auto <port-list> | blocked <port-list> | forward <port-list> ]
```
#### View IGMP Config per VLAN
```markup
show ip igmp [vlan VLAN_ID]
```
#### View IGMP Config
```markup
show ip igmp config
show ip igmp vlan VLANID config
```
#### IGMP Statistics
```markup
show ip igmp statistics
```
#### IGMP Storical Counters
```markup
show ip igmp statistics
```
#### IGMP Groups Address Information
```markup
show ip igmp groups
```
To view for specific VLAN with specified address:
```markup
show ip igmp vlan VLAN_ID group IP_ADDR
```
#### Enable IGMP v3
```markup
igmp lookup-mode ip
(config) vlan VLANID ip igmp version 3 
```
#### IGMP Debugging
```markup
debug destination buffer
debug ip igmp
show debug
show debug buffer | include TXT
```

### VSF 
---
#### Create VSF: Auto-Join / Plug and Play
Configure one switch with VSF and a second, factory default switch that is connected will join and form a VSF automatically 

**Configure Member 1** – configure one switch with VSF and reboot 
```markup
vsf member 1 link 1 b1
vsf enable domain 2
```
**Connnect Member 2** – connect a factory default switch to the VSF port configured on Member 1. 
After a few brief moments, the VSF will detect the new device, reboot the new switch and join the VSF. 


#### Create VSF: Manual configuration
Configure both VSF members manually 

Assign VSF ports to VSF link 
Enable VSF domain ID and reboot 


##### Configure Member 1 – configure member 1 with VSF and reboot 
```markup
vsf member 1 link 1 b1
vsf enable domain 2
```
##### Configure Member 2 – configure member 2 with VSF and reboot 
```markup
vsf member 2 link 1 b1
vsf enable domain 2
```
Connect VSF switches –connect member 1 and 2 configured VFS ports before member 2 finish its boot cycle and validate VSF status after reboot 

#### VSF provisioning – configure one switch with VSF, and manually provision a second switch with: 

Chassis type; called loose provision 
Chassis type and mac-address; called strict provisioning 
Connect a second member matching the provisioning 

##### Configuring Member 1 – configure one switch with VSF and reboot 
```markup
vsf enable domain 1 
switch(config)# vsf member 1 link 1 1/49,1/50
switch(config)# vsf member 1 link 2 1/51,1/52
```
On Member 1, provision Member 2 – after Member 1 reboots, provision Member 2 for either: 

Loose provision – This scenario is will allow ANY device with matching J# to join the VSF domain for this you will need to get the device J# (you can find it when you execute show running-config) 
```markup
switch(config)# vsf member 2 type jl256a 
switch(config)# vsf member 2 link 1 2/49,2/50 
switch(config)# vsf member 2 link 2 2/51,2/52 
```
Strict provision - This scenario is will only devices with matching J# + MAC to join the VSF domain for this you will need to get the device J# and MAC address (you can find them when you executing show running-config, and show system) 
```markup
switch(config)# vsf member 2 type jl256a mac-address e0071b-000002 
switch(config)# vsf member 2 link 1 2/49,2/50 
switch(config)# vsf member 2 link 2 2/51,2/52 
```
##### Connect Member 2 – connect member 2 and validate VSF status after reboot 
For Member 2 to join the stack it can either be default configuration or pre-provisioned as well 

> To avoid broadcast storms or loops in your network while configuring a VSF, it is recommended to first disconnect or disable all ports you want to add to or remove from the VSF. After you finish configuring the VSF, enable or re-connect the ports.

#### VSF Commands
```markup
show vsf 
show vsf detail 
show vsf link 
show vsf link detail 
vsf member x priority xxx -> If all members have the same priority, the member with the lowest MAC address is selected as Commander
show running-config 

#Removing and shutting down a VSF member 
vsf member <x> remove 

#Shutting down a member 
vsf member <x> shutdown 

#Commander failover to the Standby
redundancy switchover
```
#### Replace VSF Member

Shutdown or power down the affected member
```markup
vsf member <x> shutdown 
````
Physically disconnect all VSF links 

##### Auto

From the Commander, remove VSF related port/link configuration for the old member in the stack 
```markup
no vsf member 2 link 1 2/b5
```
From the Commander, remove the module (via software) that the VSF link was configured for the old member 
```markup
no module 2/b
```
From the Commander, loose (or strict, adding mac-address) provision the new member in the stack 
```markup
vsf member 2 type J9850A <optional mac-address>
```
Connect the new, factory default member, to the port where previous old member was connected 

New VSF member will reboot and join the VSF stack through plug-n-play 

##### Strict

If the member is to be replaced with a different model, the member must first be removed from the configuration
```markup
vsf member <x> remove
```
Once the switch has been shut down, disconnected, or removed from the VSF configuration (if required), use the following command to replace it in the VSF configuration and re-add VSF link port assignments (replace the type and MAC address with your desired values): 
```markup
switch(config)# vsf member 6 type jl256a mac-address e0071b-000001 
switch(config)# vsf member 6 link 1 6/49,6/50 
switch(config)# vsf member 6 link 2 6/51,6/52
```
### Recover Switch
---
```markup
erase startup-config - restore the factory default configuration using the console
copy tftp startup-config 10.1.0.12 xxxxx.cfg
show config files
startup-default primary config xxxxx.cfg
boot system flash primary
```
