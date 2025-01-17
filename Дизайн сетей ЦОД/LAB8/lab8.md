#         EVPN - маршрутизация между VRF через EVPN route-type 5

## План работы
- берем готовую конфигурацию лифов и спайнов из предыдущей лабы по multihoming-у
- создаем на всех лифах вторую VRF
- к бордер-лифам (leaf-2n и leaf-3n, которые в VPC) цепляем router-on-the-stick - asr1000v (Border), port-channel к VPC-паре
- от Border-1 делаем два point-to-point-а - один к бордер-лифам в одну VRF, второй в другую VRF
- создаем на бордер-лифах ipv4 address-family и строим на обоих point-to-point-ах eBGP ipv4 к Border-у
- на Border-е настраиваем на обоих eBGP-пирах as-override, чтобы Border передавал маршруты от одного пира (vrf CUST-1) другому (vrf CUST-2) с подменой as-path на свою AS, и наоборот
- проверяем что получилось

<p align="center">
 <img src="lab8-l3vpn-inter-vrf.jpg" alt="qr" width="150%" height="150%"/>
</p>
конфиги спайнов не меняются

## конфиг leaf-1 (изменения выделены):
<details>
 
<summary> конфиг leaf-1 (изменения выделены):</summary>

```highlight
leaf-10# sh run
hostname leaf-10

nv overlay evpn
feature ospf
feature bgp
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature lldp
feature bfd
feature nv overlay

fabric forwarding anycast-gateway-mac 1234.5678.0100
vlan 1,10,20,1000,2000
### vlan 210,220
vlan 10
  vn-segment 10
vlan 20
  vn-segment 20
vlan 210
  vn-segment 210
vlan 220
  vn-segment 220
vlan 1000
  vn-segment 1000
vlan 2000
  vn-segment 2000

vrf context CUST-1
  vni 1000
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context CUST-2
  vni 2000
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context management
hardware access-list tcam region racl 512
hardware access-list tcam region arp-ether 256 double-wide


interface Vlan1

interface Vlan10
  no shutdown
  vrf member CUST-1
  ip address 10.35.10.1/24
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member CUST-1
  ip address 10.35.20.1/24
  fabric forwarding mode anycast-gateway

interface Vlan210
  no shutdown
  vrf member CUST-2
  ip address 10.235.10.1/24
  fabric forwarding mode anycast-gateway

interface Vlan220
  no shutdown
  vrf member CUST-2
  ip address 10.235.20.1/24
  fabric forwarding mode anycast-gateway

interface Vlan1000
  no shutdown
  vrf member CUST-1
  ip forward

interface Vlan2000
  no shutdown
  vrf member CUST-2
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback0
  member vni 10
    suppress-arp
    ingress-replication protocol bgp
  member vni 20
    suppress-arp
    ingress-replication protocol bgp
  member vni 210
    suppress-arp
    ingress-replication protocol bgp
  member vni 220
    suppress-arp
    ingress-replication protocol bgp
  member vni 1000 associate-vrf
  member vni 2000 associate-vrf

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

interface Ethernet1/5
  switchport access vlan 210

interface Ethernet1/6

interface Ethernet1/7

interface Ethernet1/8

interface Ethernet1/9

interface Ethernet1/10

interface Ethernet1/11

interface Ethernet1/12

interface Ethernet1/13

interface Ethernet1/14

interface Ethernet1/15

interface Ethernet1/16

interface Ethernet1/17

interface Ethernet1/18

interface Ethernet1/19

interface Ethernet1/20

interface Ethernet1/21

interface Ethernet1/22

interface Ethernet1/23

interface Ethernet1/24

interface Ethernet1/25

interface Ethernet1/26

interface Ethernet1/27

interface Ethernet1/28

interface Ethernet1/29

interface Ethernet1/30

interface Ethernet1/31

interface Ethernet1/32

interface Ethernet1/33

interface Ethernet1/34

interface Ethernet1/35

interface Ethernet1/36

interface Ethernet1/37

interface Ethernet1/38

interface Ethernet1/39

interface Ethernet1/40

interface Ethernet1/41

interface Ethernet1/42

interface Ethernet1/43

interface Ethernet1/44

interface Ethernet1/45

interface Ethernet1/46

interface Ethernet1/47

interface Ethernet1/48

interface Ethernet1/49

interface Ethernet1/50

interface Ethernet1/51

interface Ethernet1/52

interface Ethernet1/53

interface Ethernet1/54

interface Ethernet1/55

interface Ethernet1/56

interface Ethernet1/57

interface Ethernet1/58

interface Ethernet1/59

interface Ethernet1/60

interface Ethernet1/61

interface Ethernet1/62

interface Ethernet1/63

interface Ethernet1/64

interface mgmt0
  vrf member management

interface loopback0
  ip address 10.33.10.0/32
  ip router ospf 65000 area 0.0.0.0
icam monitor scale

cli alias name i show interface status
cli alias name wr copy running start
line console
  exec-timeout 0
line vty
boot nxos bootflash:/nxos.9.3.14.bin 
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

</details>
   
## конфигурация sw-1
```
hostname nexus-sw-1
vlan 1,20

interface Ethernet1/1
  switchport mode trunk

interface Ethernet1/2
  switchport mode trunk

interface Ethernet1/3
  switchport access vlan 20
```

## проверяем разное при варианте без агрегации
### пинги между PC-3-20 и PC-1-10
```
root@PC-3-20:/home/gns3# ping 10.35.10.11
PING 10.35.10.11 (10.35.10.11): 56 data bytes
64 bytes from 10.35.10.11: seq=0 ttl=62 time=17.566 ms
64 bytes from 10.35.10.11: seq=1 ttl=62 time=6.517 ms
```
### смотрим что заблочен один линк на sw-1
```
nexus-sw-1# sh spanning-tree blockedports 

Name                 Blocked Interfaces List
-------------------- ------------------------------------
VLAN0001             Eth1/2
VLAN0020             Eth1/2

Number of blocked ports (segments) in the system : 2
```
### гасим не заблоченный, рабочий линк и смотрим на потери пакетов - ну да, 4 потерялось
```
root@PC-3-20:/home/gns3# ping 10.35.10.11
PING 10.35.10.11 (10.35.10.11): 56 data bytes
64 bytes from 10.35.10.11: seq=0 ttl=62 time=11.207 ms
64 bytes from 10.35.10.11: seq=1 ttl=62 time=11.343 ms
64 bytes from 10.35.10.11: seq=2 ttl=62 time=18.053 ms
64 bytes from 10.35.10.11: seq=3 ttl=62 time=12.569 ms
64 bytes from 10.35.10.11: seq=4 ttl=62 time=12.721 ms
64 bytes from 10.35.10.11: seq=5 ttl=62 time=13.462 ms
64 bytes from 10.35.10.11: seq=6 ttl=62 time=11.945 ms




64 bytes from 10.35.10.11: seq=11 ttl=62 time=12.567 ms
64 bytes from 10.35.10.11: seq=12 ttl=62 time=13.596 ms
64 bytes from 10.35.10.11: seq=13 ttl=62 time=8.969 ms
64 bytes from 10.35.10.11: seq=14 ttl=62 time=11.132 ms
64 bytes from 10.35.10.11: seq=15 ttl=62 time=9.867 ms
64 bytes from 10.35.10.11: seq=16 ttl=62 time=10.982 ms
64 bytes from 10.35.10.11: seq=17 ttl=62 time=8.121 ms
64 bytes from 10.35.10.11: seq=18 ttl=62 time=16.799 ms
64 bytes from 10.35.10.11: seq=19 ttl=62 time=6.211 ms
^C
--- 10.35.10.11 ping statistics ---
20 packets transmitted, 16 packets received, 20% packet loss
```
### поднимаем линк обратно и смотрим на потери - пропало аж 30 секунд
```
root@PC-3-20:/home/gns3# ping 10.35.10.11
PING 10.35.10.11 (10.35.10.11): 56 data bytes
64 bytes from 10.35.10.11: seq=0 ttl=62 time=20.423 ms
64 bytes from 10.35.10.11: seq=1 ttl=62 time=16.947 ms
64 bytes from 10.35.10.11: seq=2 ttl=62 time=22.277 ms
64 bytes from 10.35.10.11: seq=3 ttl=62 time=8.586 ms
64 bytes from 10.35.10.11: seq=4 ttl=62 time=11.559 ms
64 bytes from 10.35.10.11: seq=5 ttl=62 time=13.333 ms
64 bytes from 10.35.10.11: seq=6 ttl=62 time=12.995 ms
64 bytes from 10.35.10.11: seq=7 ttl=62 time=23.009 ms
64 bytes from 10.35.10.11: seq=8 ttl=62 time=13.416 ms
64 bytes from 10.35.10.11: seq=9 ttl=62 time=6.455 ms



64 bytes from 10.35.10.11: seq=42 ttl=62 time=10.556 ms
64 bytes from 10.35.10.11: seq=43 ttl=62 time=22.545 ms
64 bytes from 10.35.10.11: seq=44 ttl=62 time=11.351 ms
64 bytes from 10.35.10.11: seq=45 ttl=62 time=6.287 ms
64 bytes from 10.35.10.11: seq=46 ttl=62 time=6.346 ms
^C
--- 10.35.10.11 ping statistics ---
47 packets transmitted, 15 packets received, 68% packet loss
round-trip min/avg/max = 6.287/13.739/23.009 ms
```
## делаем агрегацию
### изменение конфигов на leaf-2 leaf-3 и sw-1
```
leaf-2 и leaf-3 одинаково:

interface Ethernet1/7
  switchport mode trunk
  channel-group 10 mode active
interface port-channel10
  switchport mode trunk
  vpc 10

sw-1:

interface Ethernet1/1
  switchport mode trunk
  channel-group 10 mode active

interface Ethernet1/2
  switchport mode trunk
  channel-group 10 mode active
interface port-channel10
  switchport mode trunk
```
## проверяем разное при варианте c агрегацией
### пинги между PC-3-20 и PC-1-10
```
root@PC-3-20:/home/gns3# ping 10.35.10.11
PING 10.35.10.11 (10.35.10.11): 56 data bytes
64 bytes from 10.35.10.11: seq=1 ttl=62 time=23.412 ms
64 bytes from 10.35.10.11: seq=2 ttl=62 time=10.729 ms
```
### смотрим vpc и агрегацию:
```
leaf-20# sh vpc
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 100 
Peer status                       : peer adjacency formed ok      
vPC keep-alive status             : peer is alive                 
Configuration consistency status  : success 
Per-vlan consistency status       : success                       
Type-2 consistency status         : success 
vPC role                          : secondary                     
Number of vPCs configured         : 1   
Peer Gateway                      : Enabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Enabled, timer is off.(timeout = 240s)
Delay-restore status              : Timer is off.(timeout = 300s)
Delay-restore SVI status          : Timer is off.(timeout = 300s)
Operational Layer3 Peer-router    : Enabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans    
--    ----   ------ -------------------------------------------------
1     Po100  up     1,10,20,1000                                                
         

vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
10    Po10          up     success     success               1,10,20,1000       
         
nexus-sw-1# sh port-c su
--------------------------------------------------------------------------------
Group Port-       Type     Protocol  Member Ports
      Channel
--------------------------------------------------------------------------------
10    Po10(SU)    Eth      LACP      Eth1/1(P)    Eth1/2(P) 
```
### ломаем один линк, потом возвращаем его, при этом смотрим на пинг - никаких потерь

