
### SOCKS in Explicit Proxy
---
Execute the below commands to configure socks in explicit proxy.

    # config web-proxy explicit
        set socks enable / disable
        et socks-incoming port user
    end

'socks{enable | disable}' <-----    Enable or disable (by default) the Socket Secure (SOCKS) proxy. Once enabled, use the socks-incoming-port entry to set the port number that SOCKS traffic from client web browsers will use to connect to the explicit proxy.socks-incoming-port <port>

#### TCP and UDP Socks
One way in which web proxy services differ from firewall services is the protocol type you can select. The following protocol types are available:

    ALL
    CONNECT
    FTP
    HTTP
    SOCKS-TCP
    SOCKS-UDP

To add a web proxy service go to Policy & Objects > Servicesand select Create New. Set Service Type to Explicit Proxy and configure the service as required.

To add a web proxy service from the CLI enter:
```markup
config firewall service custom
edit my-socks-service
set explicit-proxy enable
set category Web Proxy
set protocol SOCKS-TCP
set tcp-portrange 3450-3490
end    
```
### Revert Images
---
#### List Flash Partitions
```markup
Fortigate# diag sys flash list
Partition  Image                                     TotalSize(KB)  Used(KB)  Use%  Active
1          FGT3KD-6.02-FW-build1175-201110                  253871     82125   32%  No
2          FGT3KD-6.02-FW-build1190-201216                  253871     84802   33%  Yes
3          EXDB-1.00000                                   14866900    163192    1%  No
Image build at Dec 16 2020 22:40:34 for b1190
```

#### Set Boot Partition
```markup
FortiGate # exec set-next-reboot primary
Default image is changed to image# 1
```
```markup
FortiGate # exec set-next-reboot secondary
Image# 2 is already the default image.
```
### Prevent TCP Timestamp Response
---
```markup
config system global
    set tcp-option disable    <-- Default value is enable
end
```
