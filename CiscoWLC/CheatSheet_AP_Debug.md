### Debug Commands
---

#### Lightweight AP
---
```markup
# debug capwap console cli
# config t
# line console 0    
# line vty 0 4             
# exec-timeout 0
# session-timeout 0
# term length 0 
# show run
# show tech
# show logging
# debug dot11 {d0|d1} monitor addr <client_MAC-address>
# debug dot11 {d0|d1} trace print clients mgmt keys rxev txev rcv xmt txfail ba
# term mon
```
Where:

|Flag|Description|
|----|--------------|
|d0	|2.4 GHz radio (slot 0)|
|d1	|5 GHz radio (slot 1)|
|mgmt|	Trace management packets|
|ba|	Trace Block ACK information|
|rcv|	Trace received packets|
|keys|	Trace set keys|
|rxev|	Trace received events|
|txev|	Trace transmit events|
|txrad|	Trace transmit to radio|
|xmt|	Trace transmit packets|
|txfail|	Trace transmit failures|
|rates|	Trace rate changes|

Disable:
```markup
# u all
```

#### COS AP: 2800/3800 Series
---
```markup
# config ap client-trace address add <client_MAC-address>
# config ap client-trace filter all enable 
# config ap client-trace output console-log enable 
# config ap client-trace start 
# term mon
```
Disable
```markup
config ap client-trace stop
```
