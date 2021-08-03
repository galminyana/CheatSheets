### Debug Commands
---

#### Lightwaight AP
---
```markup
# debug dot11 {d0|d1} monitor addr <client_MAC-address>
# debug dot11 {d0|d1} trace print clients mgmt keys rxev txev rcv xmt txfail ba
# term mon
```
Where:
|Flag|Description|
------------------
|d0	|2.4 GHz radio (slot 0)
|d1	|5 GHz radio (slot 1)
|mgmt|	Trace management packets
|ba|	Trace Block ACK information
|rcv|	Trace received packets
|keys|	Trace set keys
|rxev|	Trace received events
|txev|	Trace transmit events
|txrad|	Trace transmit to radio
|xmt|	Trace transmit packets
|txfail|	Trace transmit failures
|rates|	Trace rate changes
