
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
