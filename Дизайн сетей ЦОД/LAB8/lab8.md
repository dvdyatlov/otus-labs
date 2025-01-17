#         EVPN - маршрутизация между VRF через EVPN route-type 5

## План работы
- берем готовую конфигурацию лифов и спайнов из предыдущей лабы по l3vpn
- делаем VPC между 2 и 3 лифом
- цепляем к 2 и 3 лифам свич sw-1, и клиента PC-3-20 к нему, как на картинке
- сначала эти два линка между лифами и sw-1 делаем без агрегирования, чтобы stp отработал - проверяем как работает при дергании линков
- потом агрегируем эти линки и смотрим еще раз

<p align="center">
 <img src="lab8-l3vpn-inter-vrf.jpg" alt="qr"/>
</p>
конфиги спайнов и leaf-1 не меняются

## изменения конфига leaf-2:
```
feature lacp
feature vpc

vrf context vpc

vpc domain 100
  peer-switch
  role priority 200
  peer-keepalive destination 100.100.100.101 source 100.100.100.100 vrf vpc
  delay restore 300
  peer-gateway
  layer3 peer-router
  auto-recovery
  delay restore interface-vlan 300
  ip arp synchronize
  
interface Ethernet1/4
  switchport mode trunk
  channel-group 100 mode active

interface Ethernet1/5
  no switchport
  vrf member vpc
  ip address 100.100.100.100/31
  no shutdown

interface Ethernet1/6
  switchport mode trunk
  channel-group 100 mode active

interface Ethernet1/7
  switchport mode trunk

interface loopback0
  ip address 10.33.100.0/32 secondary
```

## изменения конфига leaf-3:
```
feature lacp
feature vpc

vrf context vpc

vpc domain 100
  peer-switch
  role priority 100
  peer-keepalive destination 100.100.100.100 source 100.100.100.101 vrf vpc
  delay restore 300
  peer-gateway
  layer3 peer-router
  auto-recovery
  delay restore interface-vlan 300
  ip arp synchronize
  
interface Ethernet1/4
  switchport mode trunk
  channel-group 100 mode active

interface Ethernet1/5
  no switchport
  vrf member vpc
  ip address 100.100.100.101/31
  no shutdown

interface Ethernet1/6
  switchport mode trunk
  channel-group 100 mode active

interface Ethernet1/7
  switchport mode trunk

interface loopback0
  ip address 10.33.100.0/32 secondary
```
   
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

