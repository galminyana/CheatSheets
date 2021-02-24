
### SOCKS in Explicit Proxy
---
Execute the below commands to configure socks in explicit proxy.

    # config web-proxy explicit
        set socks enable / disable
        et socks-incoming port user
    end

'socks{enable | disable}' <-----    Enable or disable (by default) the Socket Secure (SOCKS) proxy. Once enabled, use the socks-incoming-port entry to set the port number that SOCKS traffic from client web browsers will use to connect to the explicit proxy.socks-incoming-port <port>
