План выполнения д/з к занятию №3 -  Основы проектирования сети:

I.		придумываем логику распределения адресного пространства

II.		рисуем схему

III.	расписываем адресацию на картинке и в табличном виде

IV.		пишем команды конфигурации устройств


I.		придумываем логику распределения адресного пространства

- первый октет - 10
- второй октет - режем по /29 на каждый ЦОД, т.е. для нашего 5-го ЦОДа во втором октете используем числа с 32 по 39, а именно:

32 - для адресов loopback-ов spine-ов
  
33 - для адресов loopback-ов leaf-ов

34 - для адресов p2p-link-ов

35-39 - резерв

- третий октет - номер spine-а или номер leaf-а в адресах loopback-ов, номер spine-а в адресах p2p-ов
- четвертый октет - в случае loopback-а - номер loopback-а на устройстве, в случае p2p - (номер leaf-а)*10
- именование loopback-ов:
1-я цифра - номер ЦОДа (закладываемся что их не более 9)
  
2-я цифра - 0 у spine-ов и 1 у leaf-ов

3 и 4 цифры - номер spine-а или leaf-а (leaf-ы нумеруются десятками - 10й,20й,30й, ...)


5-я цифра - номер loopback-а на устройстве (т.е. 0-9 - закладываемся что их не больше 10шт)


примеры:

4-й ЦОД - второй октет - 24/29

3-й spine, 1-й loopback - interface Loopback40031 ip 10.24.3.1/32

17-й leaf, 2-й loopback - interface Loopback41172 ip 10.25.170.2/32

p2p между 5-м spine и 19-м leaf - 10.26.5.190/31 (четный - 190 - spine, нечетный - 191 - leaf)

II. Картинка

<p align="center">
 <img src="LAB1.jpg" alt="qr"/>
</p>

III.	расписываем адресацию на картинке и в табличном виде

5-й ЦОД

spine1-loopback50010	10.32.1.0/32

spine1-E1				10.34.1.10/31

spine1-E2				10.34.1.20/31

spine1-E3				10.34.1.30/31


spine2-loopback50020	10.32.2.0/32

spine2-E1				10.34.2.10/31

spine2-E2				10.34.2.20/31

spine2-E3				10.34.2.30/31


leaf10-loopback51100	10.33.10.0/32
leaf1-E1				10.34.1.11/31
leaf1-E2				10.34.2.11/31

leaf20-loopback51200	10.33.20.0/32

leaf20-E1				10.34.1.21/31

leaf20-E2				10.34.2.21/31


leaf30-loopback51300	10.33.30.0/32

leaf30-E1				10.34.1.31/31

leaf30-E2				10.34.2.31/31

spine1#conf t
spine1(config)#interface loopback 50010
spine1(config-if)#ip address 10.32.1.0/32
spine1(config-if)#interface E1
spine1(config-if)#no switchport
spine1(config-if)#mtu 9216
spine1(config-if)#no shu
spine1(config-if)#ip address 10.34.1.10/31
spine1(config-if)#interface E2
spine1(config-if)#no switchport
spine1(config-if)#mtu 9216
spine1(config-if)#no shu
spine1(config-if)#ip address 10.34.1.20/31
spine1(config-if)#interface E3
spine1(config-if)#no switchport
spine1(config-if)#mtu 9216
spine1(config-if)#no shu
spine1(config-if)#ip address 10.34.1.30/31

spine2#conf t
spine2(config)#interface loopback 50020
spine2(config-if)#ip address 10.32.2.0/32
spine2(config-if)#interface E1
spine2(config-if)#no switchport
spine2(config-if)#mtu 9216
spine2(config-if)#no shu
spine2(config-if)#ip address 10.34.2.10/31
spine2(config-if)#interface E2
spine2(config-if)#no switchport
spine2(config-if)#mtu 9216
spine2(config-if)#no shu
spine2(config-if)#ip address 10.34.2.20/31
spine2(config-if)#interface E3
spine2(config-if)#no switchport
spine2(config-if)#mtu 9216
spine2(config-if)#no shu
spine2(config-if)#ip address 10.34.2.30/31

leaf10#conf t
leaf10(config)#interface loopback 51100
leaf10(config-if)#ip address 10.33.10.0/32
leaf10(config-if)#interface E1
leaf10(config-if)#no switchport
leaf10(config-if)#mtu 9216
leaf10(config-if)#no shu
leaf10(config-if)#ip address 10.34.1.11/31
leaf10(config-if)#interface E2
leaf10(config-if)#no switchport
leaf10(config-if)#mtu 9216
leaf10(config-if)#no shu
leaf10(config-if)#ip address 10.34.2.11/31

leaf20#conf t
leaf20(config)#interface loopback 51200
leaf20(config-if)#ip address 10.33.20.0/32
leaf20(config-if)#interface E1
leaf20(config-if)#no switchport
leaf20(config-if)#mtu 9216
leaf20(config-if)#no shu
leaf20(config-if)#ip address 10.34.1.21/31
leaf20(config-if)#interface E2
leaf20(config-if)#no switchport
leaf20(config-if)#mtu 9216
leaf20(config-if)#no shu
leaf20(config-if)#ip address 10.34.2.21/31

leaf30#conf t
leaf30(config)#interface loopback 51300
leaf30(config-if)#ip address 10.33.30.0/32
leaf30(config-if)#interface E1
leaf30(config-if)#no switchport
leaf30(config-if)#mtu 9216
leaf30(config-if)#no shu
leaf30(config-if)#ip address 10.34.1.31/31
leaf30(config-if)#interface E2
leaf30(config-if)#no switchport
leaf30(config-if)#mtu 9216
leaf30(config-if)#no shu
leaf30(config-if)#ip address 10.34.2.31/31
