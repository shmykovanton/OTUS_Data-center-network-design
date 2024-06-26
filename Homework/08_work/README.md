# Домашнее задание №8

## VxLAN. Routing.

### Задачи:

- Анонсируете суммарные префиксы клиентов в Overlay сеть
- Настроите маршрутизацию между клиентами через суммарный префикс
- Зафиксируете в документации - план работы, адресное пространство, схему сети, настройки сетевого оборудования

## Выполнение:

### Собранная схема сети

![](images/CLOS.png)

### Таблица адресов

| hostname          | interface |   IP/MASK    | Description        |
| :---------------: | :-------: | :----------: | :----------------: |
|  leaf-1           | Loopback1 | 10.2.0.1/32  |                    |
|  leaf-1           |  Eth1     | 10.4.1.1/31  | to-spine-1         |
|  leaf-1           |  Eth2     | 10.4.2.1/31  | to-spine-2         |
|                   |           |              |                    |
|  leaf-2           | Loopback1 | 10.2.0.2/32  |                    |
|  leaf-2           |  Eth1     | 10.4.1.3/31  | to-spine-1         |
|  leaf-2           |  Eth2     | 10.4.2.3/31  | to-spine-2         |
|                   |           |              |                    |
|  leaf-3           | Loopback1 | 10.2.0.3/32  |                    |
|  leaf-3           |  Eth1     | 10.4.1.5/31  | to-spine-1         |
|  leaf-3           |  Eth2     | 10.4.2.5/31  | to-spine-2         |
|  leaf-3           |  Eth5     | 10.4.3.0/31  | to-external-router |
|                   |           |              |                    |
|  external-router  | Loopback1 | 10.2.0.4/32  |                    |
|  leaf-3           |  Eth1     | 10.4.3.1/31  | to-leaf-3          |
|                   |           |              |                    |
|  spine-1          | Loopback1 | 10.1.1.0/32  |                    |
|  spine-1          |  Eth1     | 10.4.1.0/31  | to-leaf-1          |
|  spine-1          |  Eth2     | 10.4.1.2/31  | to-leaf-2          |
|  spine-1          |  Eth3     | 10.4.1.4/31  | to-leaf-3          |
|                   |           |              |                    |
|  spine-2          | Loopback1 | 10.1.2.0/32  |                    |
|  spine-2          |  Eth1     | 10.4.2.0/31  | to-leaf-1          |
|  spine-2          |  Eth2     | 10.4.2.2/31  | to-leaf-2          |
|  spine-2          |  Eth3     | 10.4.2.2/31  | to-leaf-3          |


### Таблица ASN

| hostname         | Loopback address |  ASN       |
| :------:         | :--------------: | :--------: |
|  leaf-1          | 10.2.0.1/32      | 4200010001 |
|  leaf-2          | 10.2.0.2/32      | 4200010002 |
|  leaf-3          | 10.2.0.3/32      | 4200010003 |
|  external-router | 10.1.2.0/32      | 4200020001 |
|  spine-1         | 10.1.1.0/32      | 4200000001 |
|  spine-2         | 10.1.2.0/32      | 4200000002 |


### Taблица адресов клиентов

| name       |     MAC           | Address        | VLAN | VNI   | Attached to |
| :--------: | :---------------: | :------------: | :--: | :---: | :---------: |
|  client-1  | 50:14:00:06:00:00 | 192.168.10.1/24|   10 | 10010 | leaf-1      |
|  client-2  | 50:14:00:07:00:00 | 192.168.10.2/24|   10 | 10010 | leaf-1      |
|  client-3  | 50:14:00:08:00:00 | 192.168.20.1/24|   20 | 10020 | leaf-2      |
|  client-4  | 50:14:00:09:00:00 | 192.168.20.2/24|   20 | 10020 | leaf-2      |

### Таблица VLAN VNI

| VLAN | VNI   | Type        |
| :--: | :---: | :---------: |
| 1000 | 10000 | L3VNI       |
|   10 | 10010 | L2VNI       |
|   20 | 10020 | L2VNI       |

### Адреса виртуальных шлюзов

| VLAN | VNI   | SAG IP            | SAG MAC           |
| :--: | :---: | :---------------: | :---------------: |
|   10 | 10010 | 192.168.10.254/24 | 00:00:00:01:01:01 |
|   20 | 10020 | 192.168.20.254/24 | 00:00:00:01:01:01 |

### Конфигурация оборудования

- #### [leaf-1](config/leaf-1.conf)

```
hostname leaf-1

ip routing
service routing protocols model multi-agent

vlan 10
exit
vlan 1000
exit

vrf instance Vrf-red
exit
ip routing vrf Vrf-red
ip virtual-router mac-address 00:00:00:01:01:01

router bgp 4200010001
  router-id 10.2.0.1
  neighbor SPINE peer group
  neighbor SPINE bfd
  neighbor SPINE send-community extended
  neighbor 10.4.1.0 remote-as 4200000001
  neighbor 10.4.1.0 peer group SPINE
  neighbor 10.4.2.0 remote-as 4200000002
  neighbor 10.4.2.0 peer group SPINE
  redistribute connected
  timers bgp 3 9
  bgp log-neighbor-changes
  maximum-paths 128

  address-family evpn
      neighbor 10.4.1.0 activate
      neighbor 10.4.2.0 activate
  exit

  vlan 10
    rd 10.2.0.1:10
    route-target export 4200010001:10010
    redistribute learned
  exit

  vlan 1000
    rd 10.2.0.1:1000
    route-target export 4200010001:10000
    route-target import 4200010002:10000
    route-target import 4200010003:10000
    redistribute learned
  exit

  vrf Vrf-red
    rd 10.2.0.1:1000
    redistribute connected
    address-family ipv4
      route-target export evpn 4200010001:10000
      route-target import evpn 4200010002:10000
      route-target import evpn 4200010003:10000
    exit
  exit
exit

interface Ethernet1
  description to-spine-1
  no switchport
  ip address 10.4.1.1/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
exit
interface Ethernet 2
  description to-spine-2
  no switchport
  ip address 10.4.2.1/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
exit
interface Ethernet 5
  description to-client-1
  switchport access vlan 10
exit
interface Ethernet 6
  description to-client-2
  switchport access vlan 10
exit
interface loopback 1
  ip address 10.2.0.1/32
exit
interface Vlan 10
  vrf Vrf-red
  ip address virtual 192.168.10.254/24
exit
interface Vxlan1
  vxlan udp-port 4789
  vxlan source-interface Loopback1
  vxlan vlan 10 vni 10010
  vxlan vrf Vrf-red vni 10000
exit
```

- #### [leaf-2](config/leaf-2.conf)

```
hostname leaf-2

ip routing
service routing protocols model multi-agent

vlan 20
exit
vlan 1000
exit

vrf instance Vrf-red
exit
ip routing vrf Vrf-red
ip virtual-router mac-address 00:00:00:01:01:01

router bgp 4200010002
  router-id 10.2.0.2
  neighbor SPINE peer group
  neighbor SPINE bfd
  neighbor SPINE send-community extended
  neighbor 10.4.1.2 remote-as 4200000001
  neighbor 10.4.1.2 peer group SPINE
  neighbor 10.4.2.2 remote-as 4200000002
  neighbor 10.4.2.2 peer group SPINE
  redistribute connected
  timers bgp 3 9
  bgp log-neighbor-changes
  maximum-paths 128

  address-family evpn
      neighbor 10.4.1.2 activate
      neighbor 10.4.2.2 activate
  exit

  vlan 20
    rd 10.2.0.2:20
    route-target export 4200010002:10020
    redistribute learned
  exit

  vlan 1000
    rd 10.2.0.2:1000
    route-target export 4200010002:10000
    route-target import 4200010001:10000
    route-target import 4200010003:10000
    redistribute learned
  exit

  vrf Vrf-red
    rd 10.2.0.2:1000
    redistribute connected
    address-family ipv4
      route-target export evpn 4200010002:10000
      route-target import evpn 4200010001:10000
      route-target import evpn 4200010003:10000
    exit
  exit

exit

interface Ethernet 1
  description to-spine-1
  no switchport
  ip address 10.4.1.3/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
exit
interface Ethernet 2
  description to-spine-2
  no switchport
  ip address 10.4.2.3/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
exit
interface Ethernet 5
  description to-client-3
  switchport access vlan 20
exit
interface Ethernet 6
  description to-client-4
  switchport access vlan 20
exit
interface loopback 1
  ip address 10.2.0.2/32
exit

interface Vlan 20
  vrf Vrf-red
  ip address virtual 192.168.20.254/24
exit

interface Vxlan1
  vxlan udp-port 4789
  vxlan source-interface Loopback1
  vxlan vlan 20 vni 10020
  vxlan vrf Vrf-red vni 10000
exit
```

- #### [leaf-3](config/leaf-3.conf)

```
hostname leaf-3

ip routing
service routing protocols model multi-agent

vlan 1000
exit

vrf instance Vrf-red
exit
ip routing vrf Vrf-red

ip prefix-list VXLAN-TO-EXT seq 10 permit 0.0.0.0/0 le 31
route-map VXLAN-TO-EXT-MAP permit 10
   match ip address prefix-list VXLAN-TO-EXT
exit

router bgp 4200010003
  router-id 10.2.0.3
  neighbor SPINE peer group
  neighbor SPINE bfd
  neighbor SPINE send-community extended
  neighbor 10.4.1.4 remote-as 4200000001
  neighbor 10.4.1.4 peer group SPINE
  neighbor 10.4.2.4 remote-as 4200000002
  neighbor 10.4.2.4 peer group SPINE

  redistribute connected
  timers bgp 3 9
  bgp log-neighbor-changes
  maximum-paths 128

  address-family evpn
    neighbor 10.4.1.4 activate
    neighbor 10.4.2.4 activate
  exit

  vrf Vrf-red
    rd 10.2.0.3:1000
    redistribute connected
    neighbor 10.4.3.1 remote-as 4200020001
    neighbor 10.4.3.1 route-map VXLAN-TO-EXT-MAP out
    address-family ipv4
      route-target export evpn 4200010003:10000
      route-target import evpn 4200010001:10000
      route-target import evpn 4200010002:10000
      neighbor 10.4.3.1 activate
    exit
  exit
exit
interface Vxlan1
  vxlan udp-port 4789
  vxlan source-interface Loopback1
  vxlan vrf Vrf-red vni 10000
exit
exit

interface Ethernet 1
  description to-spine-1
  no switchport
  ip address 10.4.1.5/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
exit
interface Ethernet 2
  description to-spine-2
  no switchport
  ip address 10.4.2.5/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
exit
interface Ethernet 5
  description to-external-router
  vrf Vrf-red
  no switchport
  ip address 10.4.3.0/31
  no shutdown
exit
interface loopback 1
  ip address 10.2.0.3/32
exit
```

- #### [external-router](config/external-router.conf)

```
hostname external-router

ip routing

router bgp 4200020001
  router-id 10.3.0.1
  neighbor LEAF peer group
  neighbor LEAF bfd
  neighbor LEAF send-community extended
  neighbor 10.4.3.0 remote-as 4200010003
  redistribute connected
  timers bgp 3 9
  bgp log-neighbor-changes
  maximum-paths 128
exit

interface Ethernet1
   description to-leaf-3
   no switchport
   ip address 10.4.3.1/31
   bfd interval 100 min-rx 100 multiplier 3
exit
interface Loopback1
   ip address 10.3.0.1/32
exit
```

- #### [spine-1](config/spine-1.conf)

```
hostname spine-1

ip routing
service routing protocols model multi-agent

router bgp 4200000001
  router-id 10.1.1.0
  neighbor LEAF peer group
  neighbor LEAF bfd
  neighbor LEAF send-community extended
  neighbor 10.4.1.1 remote-as 4200010001
  neighbor 10.4.1.1 peer group LEAF
  neighbor 10.4.1.3 remote-as 4200010002
  neighbor 10.4.1.3 peer group LEAF
  neighbor 10.4.1.5 remote-as 4200010003
  neighbor 10.4.1.5 peer group LEAF
  redistribute connected
  timers bgp 3 9
  bgp log-neighbor-changes
  maximum-paths 128

  address-family evpn
    neighbor 10.4.1.1 activate
    neighbor 10.4.1.3 activate
    neighbor 10.4.1.5 activate
  exit
exit

interface Ethernet 1
  description to-leaf-1
  no switchport
  ip address 10.4.1.0/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
exit
interface Ethernet 2
  description to-leaf-2
  no switchport
  ip address 10.4.1.2/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
exit
interface Ethernet 3
  description to-leaf-3
  no switchport
  ip address 10.4.1.4/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
exit
interface loopback 1
  ip address 10.1.1.0/32
exit
end
```

- #### [spine-2](config/spine-2.conf)

```
hostname spine-2

ip routing
service routing protocols model multi-agent

router bgp 4200000002
  router-id 10.1.2.0
  neighbor LEAF peer group
  neighbor LEAF bfd
  neighbor LEAF send-community extended
  neighbor 10.4.2.1 remote-as 4200010001
  neighbor 10.4.2.1 peer group LEAF
  neighbor 10.4.2.3 remote-as 4200010002
  neighbor 10.4.2.3 peer group LEAF
  neighbor 10.4.2.5 remote-as 4200010003
  neighbor 10.4.2.5 peer group LEAF
  redistribute connected
  timers bgp 3 9
  bgp log-neighbor-changes
  maximum-paths 128

  address-family evpn
    neighbor 10.4.2.1 activate
    neighbor 10.4.2.3 activate
    neighbor 10.4.2.5 activate
  exit
exit

interface Ethernet 1
  description to-leaf-1
  no switchport
  ip address 10.4.2.0/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
exit
interface Ethernet 2
  description to-leaf-2
  no switchport
  ip address 10.4.2.2/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
exit
interface Ethernet 3
  description to-leaf-3
  no switchport
  ip address 10.4.2.4/31
  no shutdown
  bfd interval 100 min-rx 100 multiplier 3
exit
interface loopback 1
  ip address 10.1.2.0/32
exit
```

### Проверка EVPN маршрутов

- #### leaf-1

~~~
leaf-1#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.2.0.1, local AS number 4200010001
Route status codes: s - suppressed, * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >     RD: 10.2.0.1:10 mac-ip 5014.0006.0000
                                -                     -       -       0       i
 * >     RD: 10.2.0.1:10 mac-ip 5014.0006.0000 192.168.10.1
                                -                     -       -       0       i
 * >     RD: 10.2.0.1:10 mac-ip 5014.0007.0000
                                -                     -       -       0       i
 * >     RD: 10.2.0.1:10 mac-ip 5014.0007.0000 192.168.10.2
                                -                     -       -       0       i
 * >Ec   RD: 10.2.0.2:20 mac-ip 5014.0008.0000
                                10.2.0.2              -       100     0       4200000002 4200010002 i
 *  ec   RD: 10.2.0.2:20 mac-ip 5014.0008.0000
                                10.2.0.2              -       100     0       4200000001 4200010002 i
 * >Ec   RD: 10.2.0.2:20 mac-ip 5014.0008.0000 192.168.20.1
                                10.2.0.2              -       100     0       4200000002 4200010002 i
 *  ec   RD: 10.2.0.2:20 mac-ip 5014.0008.0000 192.168.20.1
                                10.2.0.2              -       100     0       4200000001 4200010002 i
 * >Ec   RD: 10.2.0.2:20 mac-ip 5014.0009.0000
                                10.2.0.2              -       100     0       4200000001 4200010002 i
 *  ec   RD: 10.2.0.2:20 mac-ip 5014.0009.0000
                                10.2.0.2              -       100     0       4200000002 4200010002 i
 * >Ec   RD: 10.2.0.2:20 mac-ip 5014.0009.0000 192.168.20.2
                                10.2.0.2              -       100     0       4200000001 4200010002 i
 *  ec   RD: 10.2.0.2:20 mac-ip 5014.0009.0000 192.168.20.2
                                10.2.0.2              -       100     0       4200000002 4200010002 i
 * >     RD: 10.2.0.1:10 imet 10.2.0.1
                                -                     -       -       0       i
 * >Ec   RD: 10.2.0.2:20 imet 10.2.0.2
                                10.2.0.2              -       100     0       4200000002 4200010002 i
 *  ec   RD: 10.2.0.2:20 imet 10.2.0.2
                                10.2.0.2              -       100     0       4200000001 4200010002 i
 * >Ec   RD: 10.2.0.3:1000 ip-prefix 10.3.0.1/32
                                10.2.0.3              -       100     0       4200000001 4200010003 4200020001 i
 *  ec   RD: 10.2.0.3:1000 ip-prefix 10.3.0.1/32
                                10.2.0.3              -       100     0       4200000002 4200010003 4200020001 i
 * >Ec   RD: 10.2.0.3:1000 ip-prefix 10.4.3.0/31
                                10.2.0.3              -       100     0       4200000001 4200010003 i
 *  ec   RD: 10.2.0.3:1000 ip-prefix 10.4.3.0/31
                                10.2.0.3              -       100     0       4200000002 4200010003 i
 * >     RD: 10.2.0.1:1000 ip-prefix 192.168.10.0/24
                                -                     -       -       0       i
 * >Ec   RD: 10.2.0.2:1000 ip-prefix 192.168.20.0/24
                                10.2.0.2              -       100     0       4200000002 4200010002 i
 *  ec   RD: 10.2.0.2:1000 ip-prefix 192.168.20.0/24
                                10.2.0.2              -       100     0       4200000001 4200010002 i
~~~

- #### leaf-2

~~~
leaf-2#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.2.0.2, local AS number 4200010002
Route status codes: s - suppressed, * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 10.2.0.1:10 mac-ip 5014.0006.0000
                                10.2.0.1              -       100     0       4200000001 4200010001 i
 *  ec   RD: 10.2.0.1:10 mac-ip 5014.0006.0000
                                10.2.0.1              -       100     0       4200000002 4200010001 i
 * >Ec   RD: 10.2.0.1:10 mac-ip 5014.0006.0000 192.168.10.1
                                10.2.0.1              -       100     0       4200000001 4200010001 i
 *  ec   RD: 10.2.0.1:10 mac-ip 5014.0006.0000 192.168.10.1
                                10.2.0.1              -       100     0       4200000002 4200010001 i
 * >Ec   RD: 10.2.0.1:10 mac-ip 5014.0007.0000
                                10.2.0.1              -       100     0       4200000001 4200010001 i
 *  ec   RD: 10.2.0.1:10 mac-ip 5014.0007.0000
                                10.2.0.1              -       100     0       4200000002 4200010001 i
 * >Ec   RD: 10.2.0.1:10 mac-ip 5014.0007.0000 192.168.10.2
                                10.2.0.1              -       100     0       4200000001 4200010001 i
 *  ec   RD: 10.2.0.1:10 mac-ip 5014.0007.0000 192.168.10.2
                                10.2.0.1              -       100     0       4200000002 4200010001 i
 * >     RD: 10.2.0.2:20 mac-ip 5014.0008.0000
                                -                     -       -       0       i
 * >     RD: 10.2.0.2:20 mac-ip 5014.0008.0000 192.168.20.1
                                -                     -       -       0       i
 * >     RD: 10.2.0.2:20 mac-ip 5014.0009.0000
                                -                     -       -       0       i
 * >     RD: 10.2.0.2:20 mac-ip 5014.0009.0000 192.168.20.2
                                -                     -       -       0       i
 * >Ec   RD: 10.2.0.1:10 imet 10.2.0.1
                                10.2.0.1              -       100     0       4200000001 4200010001 i
 *  ec   RD: 10.2.0.1:10 imet 10.2.0.1
                                10.2.0.1              -       100     0       4200000002 4200010001 i
 * >     RD: 10.2.0.2:20 imet 10.2.0.2
                                -                     -       -       0       i
 * >Ec   RD: 10.2.0.3:1000 ip-prefix 10.3.0.1/32
                                10.2.0.3              -       100     0       4200000002 4200010003 4200020001 i
 *  ec   RD: 10.2.0.3:1000 ip-prefix 10.3.0.1/32
                                10.2.0.3              -       100     0       4200000001 4200010003 4200020001 i
 * >Ec   RD: 10.2.0.3:1000 ip-prefix 10.4.3.0/31
                                10.2.0.3              -       100     0       4200000002 4200010003 i
 *  ec   RD: 10.2.0.3:1000 ip-prefix 10.4.3.0/31
                                10.2.0.3              -       100     0       4200000001 4200010003 i
 * >Ec   RD: 10.2.0.1:1000 ip-prefix 192.168.10.0/24
                                10.2.0.1              -       100     0       4200000001 4200010001 i
 *  ec   RD: 10.2.0.1:1000 ip-prefix 192.168.10.0/24
                                10.2.0.1              -       100     0       4200000002 4200010001 i
 * >     RD: 10.2.0.2:1000 ip-prefix 192.168.20.0/24
                                -                     -       -       0       i
~~~

- #### leaf-3

~~~
leaf-3#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.2.0.3, local AS number 4200010003
Route status codes: s - suppressed, * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec   RD: 10.2.0.1:10 mac-ip 5014.0006.0000
                                10.2.0.1              -       100     0       4200000001 4200010001 i
 *  ec   RD: 10.2.0.1:10 mac-ip 5014.0006.0000
                                10.2.0.1              -       100     0       4200000002 4200010001 i
 * >Ec   RD: 10.2.0.1:10 mac-ip 5014.0006.0000 192.168.10.1
                                10.2.0.1              -       100     0       4200000002 4200010001 i
 *  ec   RD: 10.2.0.1:10 mac-ip 5014.0006.0000 192.168.10.1
                                10.2.0.1              -       100     0       4200000001 4200010001 i
 * >Ec   RD: 10.2.0.1:10 mac-ip 5014.0007.0000
                                10.2.0.1              -       100     0       4200000001 4200010001 i
 *  ec   RD: 10.2.0.1:10 mac-ip 5014.0007.0000
                                10.2.0.1              -       100     0       4200000002 4200010001 i
 * >Ec   RD: 10.2.0.1:10 mac-ip 5014.0007.0000 192.168.10.2
                                10.2.0.1              -       100     0       4200000001 4200010001 i
 *  ec   RD: 10.2.0.1:10 mac-ip 5014.0007.0000 192.168.10.2
                                10.2.0.1              -       100     0       4200000002 4200010001 i
 * >Ec   RD: 10.2.0.2:20 mac-ip 5014.0008.0000
                                10.2.0.2              -       100     0       4200000002 4200010002 i
 *  ec   RD: 10.2.0.2:20 mac-ip 5014.0008.0000
                                10.2.0.2              -       100     0       4200000001 4200010002 i
 * >Ec   RD: 10.2.0.2:20 mac-ip 5014.0008.0000 192.168.20.1
                                10.2.0.2              -       100     0       4200000002 4200010002 i
 *  ec   RD: 10.2.0.2:20 mac-ip 5014.0008.0000 192.168.20.1
                                10.2.0.2              -       100     0       4200000001 4200010002 i
 * >Ec   RD: 10.2.0.2:20 mac-ip 5014.0009.0000
                                10.2.0.2              -       100     0       4200000001 4200010002 i
 *  ec   RD: 10.2.0.2:20 mac-ip 5014.0009.0000
                                10.2.0.2              -       100     0       4200000002 4200010002 i
 * >Ec   RD: 10.2.0.2:20 mac-ip 5014.0009.0000 192.168.20.2
                                10.2.0.2              -       100     0       4200000001 4200010002 i
 *  ec   RD: 10.2.0.2:20 mac-ip 5014.0009.0000 192.168.20.2
                                10.2.0.2              -       100     0       4200000002 4200010002 i
 * >Ec   RD: 10.2.0.1:10 imet 10.2.0.1
                                10.2.0.1              -       100     0       4200000002 4200010001 i
 *  ec   RD: 10.2.0.1:10 imet 10.2.0.1
                                10.2.0.1              -       100     0       4200000001 4200010001 i
 * >Ec   RD: 10.2.0.2:20 imet 10.2.0.2
                                10.2.0.2              -       100     0       4200000002 4200010002 i
 *  ec   RD: 10.2.0.2:20 imet 10.2.0.2
                                10.2.0.2              -       100     0       4200000001 4200010002 i
 * >     RD: 10.2.0.3:1000 ip-prefix 10.3.0.1/32
                                -                     -       100     0       4200020001 i
 * >     RD: 10.2.0.3:1000 ip-prefix 10.4.3.0/31
                                -                     -       -       0       i
 *       RD: 10.2.0.3:1000 ip-prefix 10.4.3.0/31
                                -                     -       100     0       4200020001 i
 * >Ec   RD: 10.2.0.1:1000 ip-prefix 192.168.10.0/24
                                10.2.0.1              -       100     0       4200000002 4200010001 i
 *  ec   RD: 10.2.0.1:1000 ip-prefix 192.168.10.0/24
                                10.2.0.1              -       100     0       4200000001 4200010001 i
 * >Ec   RD: 10.2.0.2:1000 ip-prefix 192.168.20.0/24
                                10.2.0.2              -       100     0       4200000002 4200010002 i
 *  ec   RD: 10.2.0.2:1000 ip-prefix 192.168.20.0/24
                                10.2.0.2              -       100     0       4200000001 4200010002 i
~~~

- #### external-router

~~~
external-router#show ip route

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

 C        10.3.0.1/32 is directly connected, Loopback1
 C        10.4.3.0/31 is directly connected, Ethernet1
 B E      192.168.10.0/24 [200/0] via 10.4.3.0, Ethernet1
 B E      192.168.20.0/24 [200/0] via 10.4.3.0, Ethernet1
~~~

### Проверка доступности

- #### client-1

~~~
vyos@client-1:~$ ping 192.168.10.2
PING 192.168.10.2 (192.168.10.2) 56(84) bytes of data.
64 bytes from 192.168.10.2: icmp_seq=1 ttl=64 time=26.1 ms
64 bytes from 192.168.10.2: icmp_seq=2 ttl=64 time=7.13 ms
64 bytes from 192.168.10.2: icmp_seq=3 ttl=64 time=5.86 ms
64 bytes from 192.168.10.2: icmp_seq=4 ttl=64 time=4.81 ms
64 bytes from 192.168.10.2: icmp_seq=5 ttl=64 time=5.28 ms
^C
--- 192.168.10.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 4.806/9.845/26.146/8.187 ms
vyos@client-1:~$ ping 192.168.20.1
PING 192.168.20.1 (192.168.20.1) 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=62 time=222 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=62 time=22.2 ms
64 bytes from 192.168.20.1: icmp_seq=3 ttl=62 time=22.0 ms
64 bytes from 192.168.20.1: icmp_seq=4 ttl=62 time=31.8 ms
64 bytes from 192.168.20.1: icmp_seq=5 ttl=62 time=19.2 ms
^C
--- 192.168.20.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 19.219/63.387/221.772/79.307 ms
vyos@client-1:~$ ping 192.168.20.2
PING 192.168.20.2 (192.168.20.2) 56(84) bytes of data.
64 bytes from 192.168.20.2: icmp_seq=1 ttl=62 time=58.0 ms
64 bytes from 192.168.20.2: icmp_seq=2 ttl=62 time=20.2 ms
64 bytes from 192.168.20.2: icmp_seq=3 ttl=62 time=21.7 ms
64 bytes from 192.168.20.2: icmp_seq=4 ttl=62 time=21.6 ms
64 bytes from 192.168.20.2: icmp_seq=5 ttl=62 time=21.6 ms
^C
--- 192.168.20.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 20.217/28.633/57.999/14.693 ms
vyos@client-1:~$ ping 10.3.0.1
PING 10.3.0.1 (10.3.0.1) 56(84) bytes of data.
64 bytes from 10.3.0.1: icmp_seq=1 ttl=62 time=29.3 ms
64 bytes from 10.3.0.1: icmp_seq=2 ttl=62 time=21.9 ms
64 bytes from 10.3.0.1: icmp_seq=3 ttl=62 time=22.3 ms
64 bytes from 10.3.0.1: icmp_seq=4 ttl=62 time=19.3 ms
64 bytes from 10.3.0.1: icmp_seq=5 ttl=62 time=23.2 ms
^C
--- 10.3.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 19.325/23.205/29.258/3.291 ms

~~~

- #### client-2

~~~
vyos@client-2:~$ ping 192.168.10.1
PING 192.168.10.1 (192.168.10.1) 56(84) bytes of data.
64 bytes from 192.168.10.1: icmp_seq=1 ttl=64 time=5.74 ms
64 bytes from 192.168.10.1: icmp_seq=2 ttl=64 time=5.17 ms
64 bytes from 192.168.10.1: icmp_seq=3 ttl=64 time=4.87 ms
64 bytes from 192.168.10.1: icmp_seq=4 ttl=64 time=6.35 ms
64 bytes from 192.168.10.1: icmp_seq=5 ttl=64 time=5.96 ms
^C
--- 192.168.10.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 4.870/5.616/6.348/0.533 ms
vyos@client-2:~$ ping 192.168.20.1
PING 192.168.20.1 (192.168.20.1) 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=62 time=21.0 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=62 time=20.5 ms
64 bytes from 192.168.20.1: icmp_seq=3 ttl=62 time=39.4 ms
64 bytes from 192.168.20.1: icmp_seq=4 ttl=62 time=21.9 ms
64 bytes from 192.168.20.1: icmp_seq=5 ttl=62 time=32.6 ms
^C
--- 192.168.20.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 20.474/27.075/39.440/7.631 ms
vyos@client-2:~$ ping 192.168.20.2
PING 192.168.20.2 (192.168.20.2) 56(84) bytes of data.
64 bytes from 192.168.20.2: icmp_seq=1 ttl=62 time=28.3 ms
64 bytes from 192.168.20.2: icmp_seq=2 ttl=62 time=22.0 ms
64 bytes from 192.168.20.2: icmp_seq=3 ttl=62 time=18.9 ms
64 bytes from 192.168.20.2: icmp_seq=4 ttl=62 time=20.2 ms
64 bytes from 192.168.20.2: icmp_seq=5 ttl=62 time=20.4 ms
^C
--- 192.168.20.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 18.869/21.928/28.289/3.328 ms
vyos@client-2:~$ ping 10.3.0.1
PING 10.3.0.1 (10.3.0.1) 56(84) bytes of data.
64 bytes from 10.3.0.1: icmp_seq=1 ttl=62 time=69.1 ms
64 bytes from 10.3.0.1: icmp_seq=2 ttl=62 time=21.6 ms
64 bytes from 10.3.0.1: icmp_seq=3 ttl=62 time=19.1 ms
64 bytes from 10.3.0.1: icmp_seq=4 ttl=62 time=36.4 ms
64 bytes from 10.3.0.1: icmp_seq=5 ttl=62 time=20.5 ms
^C
--- 10.3.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4002ms
rtt min/avg/max/mdev = 19.098/33.332/69.078/18.931 ms
~~~

- #### client-3

~~~
vyos@client-3:~$ ping 192.168.10.1
PING 192.168.10.1 (192.168.10.1) 56(84) bytes of data.
64 bytes from 192.168.10.1: icmp_seq=1 ttl=62 time=39.1 ms
64 bytes from 192.168.10.1: icmp_seq=2 ttl=62 time=67.9 ms
64 bytes from 192.168.10.1: icmp_seq=3 ttl=62 time=19.3 ms
64 bytes from 192.168.10.1: icmp_seq=4 ttl=62 time=18.1 ms
64 bytes from 192.168.10.1: icmp_seq=5 ttl=62 time=75.2 ms
^C
--- 192.168.10.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 18.081/43.905/75.191/23.881 ms
vyos@client-3:~$ ping 192.168.10.2
PING 192.168.10.2 (192.168.10.2) 56(84) bytes of data.
64 bytes from 192.168.10.2: icmp_seq=1 ttl=62 time=29.2 ms
64 bytes from 192.168.10.2: icmp_seq=2 ttl=62 time=19.6 ms
64 bytes from 192.168.10.2: icmp_seq=3 ttl=62 time=32.4 ms
64 bytes from 192.168.10.2: icmp_seq=4 ttl=62 time=18.6 ms
64 bytes from 192.168.10.2: icmp_seq=5 ttl=62 time=20.1 ms
^C
--- 192.168.10.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 18.637/23.996/32.414/5.681 ms
vyos@client-3:~$ ping 192.168.20.2
PING 192.168.20.2 (192.168.20.2) 56(84) bytes of data.
64 bytes from 192.168.20.2: icmp_seq=1 ttl=64 time=6.27 ms
64 bytes from 192.168.20.2: icmp_seq=2 ttl=64 time=4.79 ms
64 bytes from 192.168.20.2: icmp_seq=3 ttl=64 time=4.92 ms
64 bytes from 192.168.20.2: icmp_seq=4 ttl=64 time=6.22 ms
64 bytes from 192.168.20.2: icmp_seq=5 ttl=64 time=5.10 ms
^C
--- 192.168.20.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 4.790/5.458/6.265/0.647 ms
vyos@client-3:~$ ping 10.3.0.1
PING 10.3.0.1 (10.3.0.1) 56(84) bytes of data.
64 bytes from 10.3.0.1: icmp_seq=1 ttl=62 time=41.1 ms
64 bytes from 10.3.0.1: icmp_seq=2 ttl=62 time=20.8 ms
64 bytes from 10.3.0.1: icmp_seq=3 ttl=62 time=24.8 ms
64 bytes from 10.3.0.1: icmp_seq=4 ttl=62 time=27.0 ms
64 bytes from 10.3.0.1: icmp_seq=5 ttl=62 time=19.0 ms
^C
--- 10.3.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 18.997/26.531/41.123/7.816 ms
~~~

- #### client-4

~~~
vyos@client-4:~$ ping 192.168.10.1
PING 192.168.10.1 (192.168.10.1) 56(84) bytes of data.
64 bytes from 192.168.10.1: icmp_seq=1 ttl=62 time=38.4 ms
64 bytes from 192.168.10.1: icmp_seq=2 ttl=62 time=20.1 ms
64 bytes from 192.168.10.1: icmp_seq=3 ttl=62 time=22.8 ms
64 bytes from 192.168.10.1: icmp_seq=4 ttl=62 time=17.6 ms
64 bytes from 192.168.10.1: icmp_seq=5 ttl=62 time=23.1 ms
^C
--- 192.168.10.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 17.558/24.380/38.356/7.270 ms
vyos@client-4:~$ ping 192.168.10.2
PING 192.168.10.2 (192.168.10.2) 56(84) bytes of data.
64 bytes from 192.168.10.2: icmp_seq=1 ttl=62 time=26.2 ms
64 bytes from 192.168.10.2: icmp_seq=2 ttl=62 time=20.7 ms
64 bytes from 192.168.10.2: icmp_seq=3 ttl=62 time=68.2 ms
64 bytes from 192.168.10.2: icmp_seq=4 ttl=62 time=19.7 ms
64 bytes from 192.168.10.2: icmp_seq=5 ttl=62 time=29.4 ms
^C
--- 192.168.10.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 19.723/32.852/68.233/18.046 ms
vyos@client-4:~$ ping 192.168.20.1
PING 192.168.20.1 (192.168.20.1) 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=64 time=5.15 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=64 time=5.24 ms
64 bytes from 192.168.20.1: icmp_seq=3 ttl=64 time=4.87 ms
64 bytes from 192.168.20.1: icmp_seq=4 ttl=64 time=8.27 ms
64 bytes from 192.168.20.1: icmp_seq=5 ttl=64 time=4.99 ms
^C
--- 192.168.20.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 4.866/5.701/8.269/1.289 ms
vyos@client-4:~$ ping 10.3.0.1
PING 10.3.0.1 (10.3.0.1) 56(84) bytes of data.
64 bytes from 10.3.0.1: icmp_seq=1 ttl=62 time=40.5 ms
64 bytes from 10.3.0.1: icmp_seq=2 ttl=62 time=18.4 ms
64 bytes from 10.3.0.1: icmp_seq=3 ttl=62 time=22.7 ms
64 bytes from 10.3.0.1: icmp_seq=4 ttl=62 time=25.6 ms
64 bytes from 10.3.0.1: icmp_seq=5 ttl=62 time=28.5 ms
^C
--- 10.3.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4008ms
rtt min/avg/max/mdev = 18.443/27.140/40.496/7.455 ms
~~~
