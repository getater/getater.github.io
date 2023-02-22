---
title: "Snippet: Default nic"
categories:
  - Snippet
tags:
  - nic
  - network card
  - default gateway
  - ipv4
---
Checks to make sure there's at least one IPv4 address, and if there is, it will find the NIC associated with the default route 0.0.0.0 using the routing table and NIC info classes in WMI.  This should be the default nic.

```sql
//get local IPv4 adapters
@ip = Network.GetIpAddresses();
@ip = SELECT * FROM @ip WHERE IpVersion = "IPv4";

//verify
IF NOT(@ip)
    NOTIMPLEMENTED "No IPv4 addresses were returned from Network.GetIpAddresses(). Either this device is only using IPv6 or this device may be in airplane mode";
ENDIF;

//get default route(s)
@route = NativeServices.RunWmiQuery(Namespace:"ROOT\\cimv2", Query:"SELECT * FROM Win32_IP4RouteTable", ResultsAsText:False);
@nic = NativeServices.RunWmiQuery(Namespace:"ROOT\\cimv2", Query:"SELECT * FROM Win32_NetworkAdapterConfiguration", ResultsAsText:False);
@route = 
    SELECT 
        ROW_NUMBER() OVER(PARTITION BY Destination ORDER BY rt.Metric1, rt.Metric2, rt.Metric3, rt.Metric4, rt.Metric5, rt.InterfaceIndex) AS InterfaceRanking,
        nic.Caption AS NicCaption,
        nic.Description AS NicDescription,
        nic.MACAddress,
        nic.ServiceName
    FROM 
        @route rt
        LEFT JOIN @nic nic
            ON rt.InterfaceIndex = nic.InterfaceIndex
    WHERE 
        Destination = "0.0.0.0";

//verify
IF NOT(@route)
    NOTIMPLEMENTED "No default route was found. This device may be in airplane mode or this was checked while networking was still starting up on a complex, multi-homed device";
ENDIF;
```
