#         l2vpn evpn и ospf в качестве underlay

## План работы
- конфигурим девайсы в соответствии с картинкой 
- проверяем доступность между loopback-ами - ospf underlay
- конфигурим все девайсы в одну bgp as (ibgp), спайны как роут-рефлекторы, на всех  девайсах делаем bgp address-family evpn, все отдают bgp extended community


<p align="center">
 <img src="LAB5-arista-nxos-l2vpn.jpg" alt="qr"/>
</p>

## конфигурация spine01
```
interface Ethernet1
   no switchport
   ip address 10.34.1.10/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   no switchport
   ip address 10.34.1.20/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   no switchport
   ip address 10.34.1.30/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 10.32.1.0/32
   ip ospf area 0.0.0.0

ip routing
!
router bgp 65000
   router-id 10.32.1.0
   bgp listen range 10.33.0.0/16 peer-group LEAVES remote-as 65000
   neighbor LEAVES peer group
   neighbor LEAVES remote-as 65000
   neighbor LEAVES update-source Loopback1
   neighbor LEAVES bfd
   neighbor LEAVES route-reflector-client
   neighbor LEAVES send-community extended
   !
   address-family evpn
      neighbor LEAVES activate
!
router ospf 65000
   max-lsa 12000
!
end
```
   
## конфигурация spine02
```
interface Ethernet1
   no switchport
   ip address 10.34.2.10/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   no switchport
   ip address 10.34.2.20/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   no switchport
   ip address 10.34.2.30/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 10.32.2.0/32
   ip ospf area 0.0.0.0
!
ip routing
!
router bgp 65000
   router-id 10.32.2.0
   bgp listen range 10.33.0.0/16 peer-group LEAVES remote-as 65000
   neighbor LEAVES peer group
   neighbor LEAVES remote-as 65000
   neighbor LEAVES update-source Loopback1
   neighbor LEAVES bfd
   neighbor LEAVES route-reflector-client
   neighbor LEAVES send-community extended
   !
   address-family evpn
      neighbor LEAVES activate
!
router ospf 65000
   max-lsa 12000
```

## конфигурация leaf10
```
hostname leaf-10
nv overlay evpn
feature ospf
feature bgp
feature fabric forwarding
feature vn-segment-vlan-based
feature bfd
feature nv overlay
vlan 1,10,20
vlan 10
  vn-segment 10
vlan 20
  vn-segment 20
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback0
  member vni 10
    ingress-replication protocol bgp
  member vni 20
    ingress-replication protocol bgp

interface Ethernet1/1
  no switchport
  ip address 10.34.1.11/31
  ip ospf network point-to-point
  ip router ospf 65000 area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 10.34.2.11/31
  ip ospf network point-to-point
  ip router ospf 65000 area 0.0.0.0
  no shutdown
interface Ethernet1/3
  switchport access vlan 10
interface Ethernet1/4
  switchport access vlan 20
interface loopback0
  ip address 10.33.10.0/32
  ip router ospf 65000 area 0.0.0.0

router ospf 65000
router bgp 65000
  address-family l2vpn evpn
  template peer SPINES
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.32.1.0
    inherit peer SPINES
  neighbor 10.32.2.0
    inherit peer SPINES
```

## конфигурация leaf20
```
hostname leaf-20
nv overlay evpn
feature ospf
feature bgp
feature fabric forwarding
feature vn-segment-vlan-based
feature bfd
feature nv overlay
vlan 1,10
vlan 10
  vn-segment 10
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback0
  member vni 10
    ingress-replication protocol bgp

interface Ethernet1/1
  no switchport
  ip address 10.34.1.21/31
  ip ospf network point-to-point
  ip router ospf 65000 area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 10.34.2.21/31
  ip ospf network point-to-point
  ip router ospf 65000 area 0.0.0.0
  no shutdown
interface Ethernet1/3
  switchport access vlan 10
interface loopback0
  ip address 10.33.20.0/32
  ip router ospf 65000 area 0.0.0.0

router ospf 65000
router bgp 65000
  address-family l2vpn evpn
  template peer SPINES
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.32.1.0
    inherit peer SPINES
  neighbor 10.32.2.0
    inherit peer SPINES
```

## конфигурация leaf30
```
hostname leaf-30
nv overlay evpn
feature ospf
feature bgp
feature fabric forwarding
feature vn-segment-vlan-based
feature bfd
feature nv overlay

vlan 1,20
vlan 20
  vn-segment 20
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback0
  member vni 20
    ingress-replication protocol bgp
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback0
  member vni 20
    ingress-replication protocol bgp

interface Ethernet1/1
  no switchport
  ip address 10.34.1.31/31
  ip ospf network point-to-point
  ip router ospf 65000 area 0.0.0.0
  no shutdown

interface Ethernet1/2
  no switchport
  ip address 10.34.2.31/31
  ip ospf network point-to-point
  ip router ospf 65000 area 0.0.0.0
  no shutdown

interface Ethernet1/3
  switchport access vlan 20
interface loopback0
  ip address 10.33.30.0/32
  ip router ospf 65000 area 0.0.0.0

router ospf 65000
router bgp 65000
  address-family l2vpn evpn
  template peer SPINES
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.32.1.0
    inherit peer SPINES
  neighbor 10.32.2.0
    inherit peer SPINES
```
## проверяем разное
### проверяем что ospf underlay поднялся и есть связность между loopback-ами
```
leaf-10# sh ip ospf neighbors 
 OSPF Process ID 65000 VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.32.1.0         0 FULL/ -          01:42:50 10.34.1.10      Eth1/1 
 10.32.2.0         0 FULL/ -          01:42:51 10.34.2.10      Eth1/2

leaf-10# ping 10.33.30.0 source 10.33.10.0
PING 10.33.30.0 (10.33.30.0) from 10.33.10.0: 56 data bytes
64 bytes from 10.33.30.0: icmp_seq=0 ttl=253 time=4.388 ms
64 bytes from 10.33.30.0: icmp_seq=1 ttl=253 time=3.89 ms
64 bytes from 10.33.30.0: icmp_seq=2 ttl=253 time=3.616 ms
64 bytes from 10.33.30.0: icmp_seq=3 ttl=253 time=3.669 ms
64 bytes from 10.33.30.0: icmp_seq=4 ttl=253 time=3.584 ms

--- 10.33.30.0 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 3.584/3.829/4.388 ms

```
### проверяем статусы bfd соседства
```
spine02#show bfd peers
VRF name: default
-----------------
DstAddr        MyDisc    YourDisc  Interface/Transport    Type          LastUp 
---------- ----------- ----------- -------------------- ------- ---------------
10.34.2.11 3909103371  2165189215        Ethernet1(11)  normal  12/02/24 00:44 
10.34.2.21 3643785541   360989875        Ethernet2(12)  normal  12/02/24 00:44 
10.34.2.31 3003998637  3562754630        Ethernet3(13)  normal  12/02/24 00:55 

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up

```
### убеждаемся что bgp использует bfd
```
spine01#sh ip bgp neighbors bfd 
BGP BFD Neighbor Table
Flags: U - BFD is enabled for BGP neighbor and BFD session state is UP
       I - BFD is enabled for BGP neighbor and BFD session state is INIT
       D - BFD is enabled for BGP neighbor and BFD session state is DOWN
       N - BFD is not enabled for BGP neighbor
Neighbor           Interface          Up/Down    State       Flags
10.34.1.11         Ethernet1          00:56:43   Established U    
10.34.1.21         Ethernet2          00:56:43   Established U    
10.34.1.31         Ethernet3          00:41:13   Established U    
```
### смотрим на таблицы маршрутов bgp, видим что маршрутов до других leaf-ов по 2 штуки
```
leaf10#sh ip bgp
BGP routing table information for VRF default
Router identifier 10.33.10.0, local AS number 65010
Route status codes: * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network                Next Hop            Metric  LocPref Weight  Path
 * >     10.33.10.0/32          -                     0       0       -       i
 * >Ec   10.33.20.0/32          10.34.1.10            0       100     0       65000 65020 i
 *  ec   10.33.20.0/32          10.34.2.10            0       100     0       65000 65020 i
 * >Ec   10.33.30.0/32          10.34.1.10            0       100     0       65000 65030 i
 *  ec   10.33.30.0/32          10.34.2.10            0       100     0       65000 65030 i
```
### смотрим на GRT, видим по 2 равноценных маршрута до других leaf-ов
```
leaf20#sh ip ro
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,

 B E      10.33.10.0/32 [200/0] via 10.34.1.20, Ethernet1
                                via 10.34.2.20, Ethernet2
 C        10.33.20.0/32 is directly connected, Loopback20
 B E      10.33.30.0/32 [200/0] via 10.34.1.20, Ethernet1
                                via 10.34.2.20, Ethernet2
 C        10.34.1.20/31 is directly connected, Ethernet1
 C        10.34.2.20/31 is directly connected, Ethernet2
```
### проверяем ip-связность между loopback-ами leaf-ов:
```
leaf10#ping 10.33.20.0 source Loopback10
PING 10.33.20.0 (10.33.20.0) from 10.33.10.0 : 72(100) bytes of data.
80 bytes from 10.33.20.0: icmp_seq=1 ttl=63 time=12.5 ms
80 bytes from 10.33.20.0: icmp_seq=2 ttl=63 time=7.34 ms
80 bytes from 10.33.20.0: icmp_seq=3 ttl=63 time=6.80 ms
80 bytes from 10.33.20.0: icmp_seq=4 ttl=63 time=6.69 ms
80 bytes from 10.33.20.0: icmp_seq=5 ttl=63 time=8.82 ms

--- 10.33.20.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 47ms
rtt min/avg/max/mdev = 6.694/8.442/12.553/2.193 ms, ipg/ewma 11.813/10.459 ms
```
Главная трудность была конечно на незнакомой аристе найти аналогичные нексусу команды про всякие peer-группы, peer-фильтры 

