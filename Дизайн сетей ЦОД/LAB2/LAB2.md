## План работы
- конфигурим девайсы в соответствии с картинкой из первой лабы
- проверяем доступность между loopback-ами


<p align="center">
 <img src="LAB1.jpg" alt="qr"/>
</p>

## конфигурация spine01

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
   
!

ip routing

!

router ospf 5

   passive-interface default
   
   no passive-interface Ethernet1
   
   no passive-interface Ethernet2
   
   no passive-interface Ethernet3
   
## конфигурация spine02

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

interface Loopback2

   ip address 10.32.2.0/32
   
   ip ospf area 0.0.0.0
   
!

ip routing

!

router ospf 5

   passive-interface default
   
   no passive-interface Ethernet1
   
   no passive-interface Ethernet2
   
   no passive-interface Ethernet3

   ## конфигурация leaf10

   interface Ethernet1
   
   no switchport
   
   ip address 10.34.1.11/31
   
   ip ospf network point-to-point
   
   ip ospf area 0.0.0.0
   
!

interface Ethernet2

   no switchport
   
   ip address 10.34.2.11/31
   
   ip ospf network point-to-point
   
   ip ospf area 0.0.0.0
   
!

interface Loopback10

   ip address 10.33.10.0/32
   
   ip ospf area 0.0.0.0
   
ip routing

!

router ospf 5

   passive-interface default
   
   no passive-interface Ethernet1
   
   no passive-interface Ethernet2

## конфигурация leaf20

interface Ethernet1

   no switchport
   
   ip address 10.34.1.21/31
   
   ip ospf network point-to-point
   
   ip ospf area 0.0.0.0
   
!

interface Ethernet2

   no switchport
   
   ip address 10.34.2.21/31
   
   ip ospf network point-to-point
   
   ip ospf area 0.0.0.0
   
!

interface Loopback20

   ip address 10.33.20.0/32
   
   ip ospf area 0.0.0.0
   
ip routing

!

router ospf 5

   passive-interface default
   
   no passive-interface Ethernet1
   
   no passive-interface Ethernet2
   
## конфигурация leaf30

interface Ethernet1

   no switchport
   
   ip address 10.34.1.31/31
   
   ip ospf network point-to-point
   
   ip ospf area 0.0.0.0
   
!

interface Ethernet2

   no switchport
   
   ip address 10.34.2.31/31
   
   ip ospf network point-to-point
   
   ip ospf area 0.0.0.0
   
!

interface Loopback30

   ip address 10.33.30.0/32
   
   ip ospf area 0.0.0.0
   
ip routing

!

router ospf 5

   passive-interface default
   
   no passive-interface Ethernet1
   
   no passive-interface Ethernet2
   
## проверяем ip-связность между loopback-ами leaf-ов:

leaf10#ping 10.33.20.0 source Loopback10

PING 10.33.20.0 (10.33.20.0) from 10.33.10.0 : 72(100) bytes of data.

80 bytes from 10.33.20.0: icmp_seq=1 ttl=63 time=16.7 ms

80 bytes from 10.33.20.0: icmp_seq=2 ttl=63 time=6.78 ms

80 bytes from 10.33.20.0: icmp_seq=3 ttl=63 time=7.74 ms

80 bytes from 10.33.20.0: icmp_seq=4 ttl=63 time=7.39 ms

80 bytes from 10.33.20.0: icmp_seq=5 ttl=63 time=8.48 ms

--- 10.33.20.0 ping statistics ---

5 packets transmitted, 5 received, 0% packet loss, time 55ms

rtt min/avg/max/mdev = 6.786/9.426/16.723/3.690 ms, pipe 2, ipg/ewma 13.847/12.981 ms




leaf10#ping 10.33.30.0 source Loopback10

PING 10.33.30.0 (10.33.30.0) from 10.33.10.0 : 72(100) bytes of data.

80 bytes from 10.33.30.0: icmp_seq=1 ttl=63 time=11.4 ms

80 bytes from 10.33.30.0: icmp_seq=2 ttl=63 time=8.39 ms

80 bytes from 10.33.30.0: icmp_seq=3 ttl=63 time=7.01 ms

80 bytes from 10.33.30.0: icmp_seq=4 ttl=63 time=7.06 ms

80 bytes from 10.33.30.0: icmp_seq=5 ttl=63 time=6.74 ms

--- 10.33.30.0 ping statistics ---

5 packets transmitted, 5 received, 0% packet loss, time 43ms

rtt min/avg/max/mdev = 6.746/8.135/11.460/1.759 ms, ipg/ewma 10.917/9.707 ms
