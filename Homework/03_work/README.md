# Домашнее задание №3

## Underlay. IS-IS

### Задачи:

- Настроите ISIS в Underlay сети, для IP связанности между всеми сетевыми устройствами.
- Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
- Убедитесь в наличии IP связанности между устройствами в ISIS домене

## Выполнение:

### Собранная схема сети

![](images/CLOS.png)

### Таблица адресов

| hostname | interface |   IP/MASK    | Description |
| :------: | :-------: | :----------: | :---------: |
|  leaf-1  | Loopback1 | 10.2.0.1/32  |             |
|  leaf-1  |  Eth1     | 10.4.1.1/31  | to-spine-1  |
|  leaf-1  |  Eth2     | 10.4.2.1/31  | to-spine-2  |
|          |           |              |             |
|  leaf-2  | Loopback1 | 10.2.0.2/32  |             |
|  leaf-2  |  Eth1     | 10.4.1.3/31  | to-spine-1  |
|  leaf-2  |  Eth2     | 10.4.2.3/31  | to-spine-2  |
|          |           |              |             |
|  leaf-3  | Loopback1 | 10.2.0.3/32  |             |
|  leaf-3  |  Eth1     | 10.4.1.5/31  | to-spine-1  |
|  leaf-3  |  Eth2     | 10.4.2.5/31  | to-spine-2  |
|          |           |              |             |
|  spine-1 | Loopback1 | 10.1.1.0/32  |             |
|  spine-1 |  Eth1     | 10.4.1.0/31  |  to-leaf-1  |
|  spine-1 |  Eth2     | 10.4.1.2/31  |  to-leaf-2  |
|  spine-1 |  Eth3     | 10.4.1.4/31  |  to-leaf-3  |
|          |           |              |             |
|  spine-2 | Loopback1 | 10.1.2.0/32  |             |
|  spine-2 |  Eth1     | 10.4.2.0/31  |  to-leaf-1  |
|  spine-2 |  Eth2     | 10.4.2.2/31  |  to-leaf-2  |
|  spine-2 |  Eth3     | 10.4.2.2/31  |  to-leaf-3  |

### Таблица ISIS ID

| hostname | Loopback address |   IS-IS ID                |
| :------: | :--------------: | :-----------------------: |
|  leaf-1  | 10.2.0.1/32      | 49.0100.0100.0200.0001.00 |
|  leaf-2  | 10.2.0.2/32      | 49.0100.0100.0200.0002.00 |
|  leaf-3  | 10.2.0.3/32      | 49.0100.0100.0200.0003.00 |
|  spine-1 | 10.1.1.0/32      | 49.0100.0100.0100.1000.00 |
|  spine-2 | 10.1.2.0/32      | 49.0100.0100.0100.2000.00 |

### Конфигурация оборудования

- #### [leaf-1](config/leaf-1.conf)

```
hostname leaf-1

ip routing

router isis 1000
   net 49.0100.0100.0200.0001.00
   router-id ipv4 10.2.0.1
   is-type level-1
   !
   address-family ipv4 unicast
      bfd all-interfaces
   exit
exit

interface Ethernet1
  description to-spine-1
  no switchport
  ip address 10.4.1.1/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
  isis enable 1000
  isis bfd
  isis hello-interval 3
  isis hello-multiplier 3
exit
interface Ethernet 2
  description to-spine-2
  no switchport
  ip address 10.4.2.1/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
  isis enable 1000
  isis bfd
  isis hello-interval 3
  isis hello-multiplier 3
exit
interface loopback 1
  ip address 10.2.0.1/32
exit
```

- #### [leaf-2](config/leaf-2.conf)

```
hostname leaf-2

ip routing

router isis 1000
   net 49.0100.0100.0200.0002.00
   router-id ipv4 10.2.0.2
   is-type level-1
   !
   address-family ipv4 unicast
      bfd all-interfaces
   exit
exit

interface Ethernet 1
  description to-spine-1
  no switchport
  ip address 10.4.1.3/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
  isis enable 1000
  isis bfd
  isis hello-interval 3
  isis hello-multiplier 3
exit
interface Ethernet 2
  description to-spine-2
  no switchport
  ip address 10.4.2.3/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
  isis enable 1000
  isis bfd
  isis hello-interval 3
  isis hello-multiplier 3
exit
interface loopback 1
  ip address 10.2.0.2/32
exit
```

- #### [leaf-3](config/leaf-3.conf)

```
hostname leaf-3

ip routing

router isis 1000
   net 49.0100.0100.0200.0003.00
   router-id ipv4 10.2.0.3
   is-type level-1
   !
   address-family ipv4 unicast
      bfd all-interfaces
   exit
exit

interface Ethernet 1
  description to-spine-1
  no switchport
  ip address 10.4.1.5/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
  isis enable 1000
  isis bfd
  isis hello-interval 3
  isis hello-multiplier 3
exit
interface Ethernet 2
  description to-spine-2
  no switchport
  ip address 10.4.2.5/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
  isis enable 1000
  isis bfd
  isis hello-interval 3
  isis hello-multiplier 3
exit
interface loopback 1
  ip address 10.2.0.3/32
exit
```

- #### [spine-1](config/spine-1.conf)

```
hostname spine-1

ip routing

router isis 1000
   net 49.0100.0100.0100.1000.00
   router-id ipv4 10.1.1.0
   is-type level-1-2
   !
   address-family ipv4 unicast
      bfd all-interfaces
   exit
exit

interface Ethernet 1
  description to-leaf-1
  no switchport
  ip address 10.4.1.0/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
  isis enable 1000
  isis bfd
  isis hello-interval 3
  isis hello-multiplier 3
exit
interface Ethernet 2
  description to-leaf-2
  no switchport
  ip address 10.4.1.2/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
  isis enable 1000
  isis bfd
  isis hello-interval 3
  isis hello-multiplier 3
exit
interface Ethernet 3
  description to-leaf-3
  no switchport
  ip address 10.4.1.4/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
  isis enable 1000
  isis bfd
  isis hello-interval 3
  isis hello-multiplier 3
exit
interface loopback 1
  ip address 10.1.1.0/32
```

- #### [spine-2](config/spine-2.conf)

```
hostname spine-2

ip routing

router isis 1000
   net 49.0100.0100.0100.2000.00
   router-id ipv4 10.1.2.0
   is-type level-1-2
   !
   address-family ipv4 unicast
      bfd all-interfaces
   exit
exit

interface Ethernet 1
  description to-leaf-1
  no switchport
  ip address 10.4.2.0/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
  isis enable 1000
  isis bfd
  isis hello-interval 3
  isis hello-multiplier 3
exit
interface Ethernet 2
  description to-leaf-2
  no switchport
  ip address 10.4.2.2/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
  isis enable 1000
  isis bfd
  isis hello-interval 3
  isis hello-multiplier 3
exit
interface Ethernet 3
  description to-leaf-3
  no switchport
  ip address 10.4.2.4/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
  isis enable 1000
  isis bfd
  isis hello-interval 3
  isis hello-multiplier 3
exit
interface loopback 1
  ip address 10.1.2.0/32
exit
```

### Проверка ISIS

- #### leaf-1

~~~
leaf-1#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
1000      default  spine-1          L1   Ethernet1          50:14:0:26:71:4a  UP    8           leaf-1.0b
1000      default  spine-1          L2   Ethernet1          50:14:0:26:71:4a  UP    8           leaf-1.0b
1000      default  spine-2          L1   Ethernet2          50:14:0:b7:d4:aa  UP    3           spine-2.0b
1000      default  spine-2          L2   Ethernet2          50:14:0:b7:d4:aa  UP    3           spine-2.0b
~~~

- #### leaf-2

~~~
leaf-2#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
1000      default  spine-1          L1   Ethernet1          50:14:0:26:71:4a  UP    8           leaf-2.0b
1000      default  spine-2          L1   Ethernet2          50:14:0:b7:d4:aa  UP    3           spine-2.0c
~~~

- #### leaf-3

~~~
leaf-3#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
1000      default  spine-1          L1   Ethernet1          50:14:0:26:71:4a  UP    7           leaf-3.0b
1000      default  spine-2          L1   Ethernet2          50:14:0:b7:d4:aa  UP    2           spine-2.0d
~~~

- #### spine-1

~~~
spine-1#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
1000      default  leaf-1           L1   Ethernet1          50:14:0:6c:75:8   UP    2           leaf-1.0b
1000      default  leaf-1           L2   Ethernet1          50:14:0:6c:75:8   UP    2           leaf-1.0b
1000      default  leaf-2           L1   Ethernet2          50:14:0:94:4e:50  UP    2           leaf-2.0b
1000      default  leaf-3           L1   Ethernet3          50:14:0:39:d9:a   UP    3           leaf-3.0b
~~~

- #### spine-2

~~~
spine-2#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
1000      default  leaf-1           L1   Ethernet1          50:14:0:6c:75:8   UP    6           spine-2.0b
1000      default  leaf-1           L2   Ethernet1          50:14:0:6c:75:8   UP    8           spine-2.0b
1000      default  leaf-2           L1   Ethernet2          50:14:0:94:4e:50  UP    8           spine-2.0c
1000      default  leaf-3           L1   Ethernet3          50:14:0:39:d9:a   UP    7           spine-2.0d
~~~


### Проверка таблиц маршрутизации

- #### leaf-1

~~~
leaf-1#show ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked

Gateway of last resort is not set

 I L1     10.1.1.0/32 [115/20] via 10.4.1.0, Ethernet1
 I L1     10.1.2.0/32 [115/20] via 10.4.2.0, Ethernet2
 C        10.2.0.1/32 is directly connected, Loopback1
 I L1     10.2.0.2/32 [115/30] via 10.4.1.0, Ethernet1
                               via 10.4.2.0, Ethernet2
 I L1     10.2.0.3/32 [115/30] via 10.4.1.0, Ethernet1
                               via 10.4.2.0, Ethernet2
 C        10.4.1.0/31 is directly connected, Ethernet1
 I L1     10.4.1.2/31 [115/20] via 10.4.1.0, Ethernet1
 I L1     10.4.1.4/31 [115/20] via 10.4.1.0, Ethernet1
 C        10.4.2.0/31 is directly connected, Ethernet2
 I L1     10.4.2.2/31 [115/20] via 10.4.2.0, Ethernet2
 I L1     10.4.2.4/31 [115/20] via 10.4.2.0, Ethernet2
~~~

- #### leaf-2

~~~
leaf-2#show ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked

Gateway of last resort is not set

 I L1     10.1.1.0/32 [115/20] via 10.4.1.2, Ethernet1
 I L1     10.1.2.0/32 [115/20] via 10.4.2.2, Ethernet2
 I L1     10.2.0.1/32 [115/30] via 10.4.1.2, Ethernet1
                               via 10.4.2.2, Ethernet2
 C        10.2.0.2/32 is directly connected, Loopback1
 I L1     10.2.0.3/32 [115/30] via 10.4.1.2, Ethernet1
                               via 10.4.2.2, Ethernet2
 I L1     10.4.1.0/31 [115/20] via 10.4.1.2, Ethernet1
 C        10.4.1.2/31 is directly connected, Ethernet1
 I L1     10.4.1.4/31 [115/20] via 10.4.1.2, Ethernet1
 I L1     10.4.2.0/31 [115/20] via 10.4.2.2, Ethernet2
 C        10.4.2.2/31 is directly connected, Ethernet2
 I L1     10.4.2.4/31 [115/20] via 10.4.2.2, Ethernet2
~~~

- #### leaf-3

~~~
leaf-3#show ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked

Gateway of last resort is not set

 I L1     10.1.1.0/32 [115/20] via 10.4.1.4, Ethernet1
 I L1     10.1.2.0/32 [115/20] via 10.4.2.4, Ethernet2
 I L1     10.2.0.1/32 [115/30] via 10.4.1.4, Ethernet1
                               via 10.4.2.4, Ethernet2
 I L1     10.2.0.2/32 [115/30] via 10.4.1.4, Ethernet1
                               via 10.4.2.4, Ethernet2
 C        10.2.0.3/32 is directly connected, Loopback1
 I L1     10.4.1.0/31 [115/20] via 10.4.1.4, Ethernet1
 I L1     10.4.1.2/31 [115/20] via 10.4.1.4, Ethernet1
 C        10.4.1.4/31 is directly connected, Ethernet1
 I L1     10.4.2.0/31 [115/20] via 10.4.2.4, Ethernet2
 I L1     10.4.2.2/31 [115/20] via 10.4.2.4, Ethernet2
 C        10.4.2.4/31 is directly connected, Ethernet2
~~~

- #### spine-1

~~~
spine-1#show ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked

Gateway of last resort is not set

 C        10.1.1.0/32 is directly connected, Loopback1
 I L1     10.1.2.0/32 [115/30] via 10.4.1.1, Ethernet1
                               via 10.4.1.3, Ethernet2
                               via 10.4.1.5, Ethernet3
 I L1     10.2.0.1/32 [115/20] via 10.4.1.1, Ethernet1
 I L1     10.2.0.2/32 [115/20] via 10.4.1.3, Ethernet2
 I L1     10.2.0.3/32 [115/20] via 10.4.1.5, Ethernet3
 C        10.4.1.0/31 is directly connected, Ethernet1
 C        10.4.1.2/31 is directly connected, Ethernet2
 C        10.4.1.4/31 is directly connected, Ethernet3
 I L1     10.4.2.0/31 [115/20] via 10.4.1.1, Ethernet1
 I L1     10.4.2.2/31 [115/20] via 10.4.1.3, Ethernet2
 I L1     10.4.2.4/31 [115/20] via 10.4.1.5, Ethernet3
~~~

- #### spine-2

~~~
spine-2#show ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - BGP, B I - iBGP, B E - eBGP,
       R - RIP, I L1 - IS-IS level 1, I L2 - IS-IS level 2,
       O3 - OSPFv3, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route, V - VXLAN Control Service,
       DH - DHCP client installed default route, M - Martian,
       DP - Dynamic Policy Route, L - VRF Leaked

Gateway of last resort is not set

 I L1     10.1.1.0/32 [115/30] via 10.4.2.1, Ethernet1
                               via 10.4.2.3, Ethernet2
                               via 10.4.2.5, Ethernet3
 C        10.1.2.0/32 is directly connected, Loopback1
 I L1     10.2.0.1/32 [115/20] via 10.4.2.1, Ethernet1
 I L1     10.2.0.2/32 [115/20] via 10.4.2.3, Ethernet2
 I L1     10.2.0.3/32 [115/20] via 10.4.2.5, Ethernet3
 I L1     10.4.1.0/31 [115/20] via 10.4.2.1, Ethernet1
 I L1     10.4.1.2/31 [115/20] via 10.4.2.3, Ethernet2
 I L1     10.4.1.4/31 [115/20] via 10.4.2.5, Ethernet3
 C        10.4.2.0/31 is directly connected, Ethernet1
 C        10.4.2.2/31 is directly connected, Ethernet2
 C        10.4.2.4/31 is directly connected, Ethernet3
~~~

### Проверка доступности

- #### spine-1

~~~
spine-1#ping 10.1.2.0
PING 10.1.2.0 (10.1.2.0) 72(100) bytes of data.
80 bytes from 10.1.2.0: icmp_seq=1 ttl=63 time=12.0 ms
80 bytes from 10.1.2.0: icmp_seq=2 ttl=63 time=11.1 ms
80 bytes from 10.1.2.0: icmp_seq=3 ttl=63 time=7.63 ms
80 bytes from 10.1.2.0: icmp_seq=4 ttl=63 time=6.73 ms
80 bytes from 10.1.2.0: icmp_seq=5 ttl=63 time=7.32 ms

--- 10.1.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 46ms
rtt min/avg/max/mdev = 6.736/8.968/12.002/2.167 ms, pipe 2, ipg/ewma 11.699/10.351 ms
spine-1#ping 10.2.0.1
PING 10.2.0.1 (10.2.0.1) 72(100) bytes of data.
80 bytes from 10.2.0.1: icmp_seq=1 ttl=64 time=5.33 ms
80 bytes from 10.2.0.1: icmp_seq=2 ttl=64 time=5.17 ms
80 bytes from 10.2.0.1: icmp_seq=3 ttl=64 time=6.54 ms
80 bytes from 10.2.0.1: icmp_seq=4 ttl=64 time=7.71 ms
80 bytes from 10.2.0.1: icmp_seq=5 ttl=64 time=5.41 ms

--- 10.2.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 25ms
rtt min/avg/max/mdev = 5.173/6.036/7.718/0.974 ms, ipg/ewma 6.423/5.706 ms
spine-1#ping 10.2.0.2
PING 10.2.0.2 (10.2.0.2) 72(100) bytes of data.
80 bytes from 10.2.0.2: icmp_seq=1 ttl=64 time=4.69 ms
80 bytes from 10.2.0.2: icmp_seq=2 ttl=64 time=5.59 ms
80 bytes from 10.2.0.2: icmp_seq=3 ttl=64 time=5.34 ms
80 bytes from 10.2.0.2: icmp_seq=4 ttl=64 time=7.67 ms
80 bytes from 10.2.0.2: icmp_seq=5 ttl=64 time=5.53 ms

--- 10.2.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 26ms
rtt min/avg/max/mdev = 4.691/5.766/7.676/1.006 ms, ipg/ewma 6.616/5.260 ms
spine-1#ping 10.2.0.3
PING 10.2.0.3 (10.2.0.3) 72(100) bytes of data.
80 bytes from 10.2.0.3: icmp_seq=1 ttl=64 time=4.04 ms
80 bytes from 10.2.0.3: icmp_seq=2 ttl=64 time=5.38 ms
80 bytes from 10.2.0.3: icmp_seq=3 ttl=64 time=6.20 ms
80 bytes from 10.2.0.3: icmp_seq=4 ttl=64 time=6.45 ms
80 bytes from 10.2.0.3: icmp_seq=5 ttl=64 time=5.67 ms

--- 10.2.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 22ms
rtt min/avg/max/mdev = 4.044/5.551/6.456/0.847 ms, ipg/ewma 5.747/4.830 ms
~~~

- #### spine-2

~~~
spine-2#ping 10.1.1.0
PING 10.1.1.0 (10.1.1.0) 72(100) bytes of data.
80 bytes from 10.1.1.0: icmp_seq=1 ttl=63 time=7.60 ms
80 bytes from 10.1.1.0: icmp_seq=2 ttl=63 time=8.46 ms
80 bytes from 10.1.1.0: icmp_seq=3 ttl=63 time=6.30 ms
80 bytes from 10.1.1.0: icmp_seq=4 ttl=63 time=10.0 ms
80 bytes from 10.1.1.0: icmp_seq=5 ttl=63 time=8.15 ms

--- 10.1.1.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 46ms
rtt min/avg/max/mdev = 6.308/8.108/10.009/1.208 ms, ipg/ewma 11.670/7.884 ms
spine-2#
spine-2#ping 10.2.0.1
PING 10.2.0.1 (10.2.0.1) 72(100) bytes of data.
80 bytes from 10.2.0.1: icmp_seq=1 ttl=64 time=5.28 ms
80 bytes from 10.2.0.1: icmp_seq=2 ttl=64 time=6.79 ms
80 bytes from 10.2.0.1: icmp_seq=3 ttl=64 time=6.83 ms
80 bytes from 10.2.0.1: icmp_seq=4 ttl=64 time=5.06 ms
80 bytes from 10.2.0.1: icmp_seq=5 ttl=64 time=4.59 ms

--- 10.2.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 25ms
rtt min/avg/max/mdev = 4.597/5.715/6.833/0.926 ms, ipg/ewma 6.311/5.447 ms
spine-2#ping 10.2.0.2
PING 10.2.0.2 (10.2.0.2) 72(100) bytes of data.
80 bytes from 10.2.0.2: icmp_seq=1 ttl=64 time=3.92 ms
80 bytes from 10.2.0.2: icmp_seq=2 ttl=64 time=7.34 ms
80 bytes from 10.2.0.2: icmp_seq=3 ttl=64 time=6.39 ms
80 bytes from 10.2.0.2: icmp_seq=4 ttl=64 time=4.82 ms
80 bytes from 10.2.0.2: icmp_seq=5 ttl=64 time=3.80 ms

--- 10.2.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 28ms
rtt min/avg/max/mdev = 3.805/5.258/7.341/1.394 ms, ipg/ewma 7.169/4.528 ms
spine-2#ping 10.2.0.3
PING 10.2.0.3 (10.2.0.3) 72(100) bytes of data.
80 bytes from 10.2.0.3: icmp_seq=1 ttl=64 time=4.04 ms
80 bytes from 10.2.0.3: icmp_seq=2 ttl=64 time=6.19 ms
80 bytes from 10.2.0.3: icmp_seq=3 ttl=64 time=5.01 ms
80 bytes from 10.2.0.3: icmp_seq=4 ttl=64 time=4.73 ms
80 bytes from 10.2.0.3: icmp_seq=5 ttl=64 time=3.35 ms

--- 10.2.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 22ms
rtt min/avg/max/mdev = 3.359/4.668/6.195/0.958 ms, ipg/ewma 5.641/4.306 ms
~~~
