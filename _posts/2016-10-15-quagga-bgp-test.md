---
layout: post
title: "Use quagga for BGP speaker"
description: "BGP Test"
catagory: Test
tags: BGP
---

[Reference](http://www.educity.cn/linux/512970.html)

# Install

```Bash:
apt-get install quagga
```

# Configuration

将 /usr/share/doc/quagga/examples 里的zedra和bgp复制到 /etc/quagga 并将后缀sample去掉 ，参考修改bgp.conf

Service quagga status

quagga支持vpnv4，不支持ls（link state）属性。

登录控制台：

```Bash:
telnet localhost 2605
```

查看配置：

```Bash:
bgpd# show  running-config    

Current configuration:
!
hostname bgpd
password zebra
log file /var/log/quagga/bgpd.log
log stdout
!
router bgp 100
 bgp router-id 10.0.2.28
 network 10.0.2.0/24
 neighbor 10.0.2.27 remote-as 100
 neighbor 10.0.2.27 next-hop-self
 neighbor 10.0.2.27 route-map set-nexthop out
!
access-list all permit any
!
route-map set-nexthop permit 10
 match ip address all
 set ip next-hop 10.0.2.28
!
line vty
!
end
bgpd# show 
  bgp             BGP information
  debugging       Debugging functions (see also 'undebug')
  history         Display the session command history
  ip              IP information
  ipv6            IPv6 information
  logging         Show current logging configuration
  memory          Memory statistics
  route-map       route-map information
  running-config  running configuration
  startup-config  Contentes of startup configuration
  thread          Thread information
  version         Displays zebra version
  work-queues     Work Queue information
bgpd# show bgp     
No BGP network exists
bgpd# show bgp 
  ipv4            Address family
  ipv6            Address family
  X:X::X:X        Network in the BGP routing table to display
  X:X::X:X/M      IPv6 prefix <network>/<length>
  community       Display routes matching the communities
  community-list  Display routes matching the community-list
  filter-list     Display routes conforming to the filter-list
  memory          Global BGP memory statistics
  neighbors       Detailed information on TCP and BGP neighbor connections
  prefix-list     Display routes conforming to the prefix-list
  regexp          Display routes matching the AS path regular expression
  route-map       Display routes matching the route-map
  rsclient        Information about Route Server Client
  summary         Summary of BGP neighbor status
  view            BGP view
  views           Show the defined BGP views
  <cr>            
bgpd# show bgp ne
bgpd# show bgp neighbors 
BGP neighbor is 10.0.2.27, remote AS 100, local AS 100, internal link
  BGP version 4, remote router ID 10.0.2.27
  BGP state = Established, up for 00:05:30
  Last read 00:00:30, hold time is 180, keepalive interval is 60 seconds
  Neighbor capabilities:
    4 Byte AS: advertised and received
    Route refresh: advertised and received(old & new)
    Address family IPv4 Unicast: advertised and received
  Message statistics:
    Inq depth is 0
    Outq depth is 0
                         Sent       Rcvd
    Opens:                  9          1
    Notifications:          0          8
    Updates:                1          1
    Keepalives:             7          6
    Route Refresh:          0          0
    Capability:             0          0
    Total:                 17         16
  Minimum time between advertisement runs is 5 seconds

 For address family: IPv4 Unicast
  NEXT_HOP is always this router
  Community attribute sent to this neighbor(both)
  Outbound path policy configured
  Route map for outgoing advertisements is *set-nexthop
  1 accepted prefixes

  Connections established 1; dropped 0
  Last reset never
Local host: 10.0.2.28, Local port: 48160
Foreign host: 10.0.2.27, Foreign port: 179
Nexthop: 10.0.2.28
Nexthop global: fe80::a00:27ff:fe23:1004
Nexthop local: ::
BGP connection: non shared network
Read thread: on  Write thread: off 
```

##Bgp.conf

```Bash:
! -*- bgp -*-
!
! BGPd sample configuratin file
!
! $Id: bgpd.conf.sample,v 1.1 2002/12/13 20:15:28 paul Exp $
!
hostname bgpd
password zebra
!enable password please-set-at-here
!
!bgp mulitple-instance
!
router bgp 100
 bgp router-id 10.0.2.28
 network 10.0.2.0/24
 neighbor 10.0.2.27 remote-as 100
 neighbor 10.0.2.27 route-map set-nexthop out
 neighbor 10.0.2.27 next-hop-self

 access-list all permit any

route-map set-nexthop permit 10
 match ip address all
 set ip next-hop 10.0.2.28

log file /var/log/quagga/bgpd.log

log stdout
```

##zebra.conf

```Bash:
! -*- zebra -*-
!
! zebra sample configuration file
!
! $Id: zebra.conf.sample,v 1.1 2002/12/13 20:15:30 paul Exp $
!
hostname Router
password zebra
enable password zebra
!
! Interface's description.
!
!interface lo
! description test of desc.
!
!interface sit0
! multicast

!
! Static default route sample.
!
!ip route 0.0.0.0/0 203.181.89.241
!

!log file /var/log/quagga/zebra.log
```

##daemons

```Bash:
zebra=yes
bgpd=yes
ospfd=no
ospf6d=no
ripd=no
ripngd=no
isisd=no
babeld=no
```






