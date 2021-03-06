#  F**king Command

* [x] reload
* [x] IPv6 initial
* [ ] password recovery
* [x] fix clock rate
* [x] fix ssh and add acl on them
* [x] fix access-list
* [x] fix dhcp which relate to access-list
* [x] check whether need to shutdown unused port
* [x] do we need ip domain-name ?
* [x] need to check whether the switches we use in the class have extra types of interfaces
* [x] Port security is required on all access ports, with a maximum of one MAC Address per port. Any violation should shut down the port.
* [x] whether vlan on switches need to add ipv6 addr
* [x] ipv6 on switches?

# Task
1. CITY Router, CITY1_SW, PC admin (Steven)
2. GLEBE Router, GLEBE_SW, PC VLAN 20 host (Zhuang)
3. CHATS Router, CHATS_SW, VLAN 10 host (on CHATS) (Tri)
4. ISP Router, CITY2_SW, VLAN 10 host on CITY2_SW (Share)

# Shortcut
`Ctr - A` : move to beginning

`Ctr - E` : move to end

`Ctr - W` : del previous word

## ISP
```
! write erase
w e
prefer dual-ipv4-and-ipv6 default
rel

conf t
h ISP
ipv u
no ip domain-l

ena s class

in Loopback0
description Loop back
ip addr 11.11.11.11 255.255.255.255
ipv6 addr 2001:11:11:11::11/128
no sh

in s0/0/0
description Link to CITY
ip addr 50.80.120.17 255.255.255.252
ipv6 addr 2001:50:80:120::/64
clock rate 128000
no shut

int s0/0/1
description Link to GLEBE
ip addr 50.80.120.21 255.255.255.252
ipv6 addr 2001:50:80:121::/64
clock rate 128000
no shut

ip route 50.80.120.0 255.255.255.248 Serial0/0/0 
ip route 50.80.120.8 255.255.255.248 Serial0/0/1 

ipv6 route 2001:50:80:100::/56 Serial0/0/0 2001:50:80:120::2
ipv6 route 2001:50:80:100::/56 Serial0/0/1 2001:50:80:121::2 5

banner motd #Authorized access only!#

ip domain-name ccna-lab.com
! in lab, use crypto key generate rsa modulus 1024
cr k g r
y
1024
username CaseStudy privilege 15 secret cisco1

line vty 0 4
exec-timeout 10
transport input ssh
login local 
logging synchronous
end
show ip ssh

conf t
line con 0
exec-timeout 10
password cisco
login
logging synchronous
exit
```

## CITY_RT
```
! enable
en
! write erase == erase startup-config
w e
prefer dual-ipv4-and-ipv6 default
! reload
rel

en
conf t
! hostname CITY
h CITY
! ipv6 unicast-routing
ipv u
! no ip domain-lookup
no ip domain-lo
! enable secret class
ena s class
! service password-encryption 
ser p
! banner motd #
ba m #Authorized access only!#

in s0/0/0
! description 'Link to ISP'
des Link to ISP
ip ad 50.80.120.18 255.255.255.252
ipv ad 2001:50:80:120::1/64
! ip nat outside
ip n o
no sh

in s0/0/1
des Link to CHATS
ip ad 172.22.14.5 255.255.255.252
ipv ad 2001:50:80:10C::/64
! ip nat inside
ip n i
clock rate 128000
no shut

in g0/0
! description 'Link to GLEBE'
des Link to GLEBE
ip ad 172.22.14.1 255.255.255.252
ipv ad 2001:50:80:10D::/64
! no shutdown
no sh

in g0/1.10
des VLAN 10 gateway
! encapsulation dot1Q 10
en d 10
ip ad 172.22.6.1 255.255.255.0
ipv ad 2001:50:80:102::/64
ip nat inside

in g0/1.20
des VLAN 20 gateway
en d 20
ip ad 172.22.0.1 255.255.252.0
ipv ad 2001:50:80:100::/64
! DHCP
ip helper-address 172.22.14.6
ip nat inside

int g0/1.30
des VLAN 30 gateway
en d 30
ip ad 172.22.4.1 255.255.254.0
ipv ad 2001:50:80:101::/64
! DHCP
ip helper-address 172.22.14.6
ip nat inside

int g0/1.137
des Native VLAN 137 gateway
! encapsulation dot1Q 137 native
en d 137 n
ip ad 172.22.7.1 255.255.255.240
ipv ad 2001:50:80:103::/64

in g0/1
no shut

int Vlan1
sh

# routing
! router rip
ro r
! version 2
v 2
! redistribute static 
r s
! passive-interface g0/1
p g0/1
p s0/0/0
p s0/0/1
! network 172.22.0.0
ne 172.22.0.0
! no auto-summary
no a

ip route 0.0.0.0 0.0.0.0 s0/0/0
ip route 172.22.12.0 255.255.252.0 s0/0/1
! ipv6 route
ipv6 route ::/0 s0/0/0 2001:50:80:120::
ipv6 route ::/0 g0/0 2001:50:80:10D::1 5
ipv6 route 2001:50:80:108::/62 s0/0/1 2001:50:80:10C::1
ipv6 route 2001:50:80:104::/62 g0/0 2001:50:80:10D::1
! access-list 1 permit host 172.22.6.254
ac 1 p h 172.22.6.254
! deny any
ac 1 d a
ac 2 p 172.22.6.0 0.0.0.255
ac 2 p 172.22.0.0 0.0.3.255
ac 2 p 172.22.4.0 0.0.1.255
ac 2 p 172.22.10.0 0.0.0.31
ac 2 p 172.22.8.0 0.0.0.255
ac 2 p 172.22.9.0 0.0.0.255
ac 2 p 172.22.13.0 0.0.0.31
ac 2 p 172.22.12.0 0.0.0.127
ac 2 p 172.22.12.128 0.0.0.127
ac 2 d a

! ip domain-name ccna-lab.com
ip domain-n ccna-lab.com
! in lab, use crypto key generate rsa modulus 1024
cr k g r
1024
! username CaseStudy privilege 15 secret cisco1
u CaseStudy pr 15 s cisco1

! line vty
line v 0 4
! access-class 1 in
acce 1 in
! exec-timeout 10
exe 10
! transport input ssh
t i s
! login local
login l 
! logging synchronous
logg s
end

conf t
line con 0
exec-timeout 10
password cisco
login
logging synchronous
end

conf t
! ip ssh time-out 75
ip s t 
! ip ssh authentication-retries 2
ip s a 2

ip nat pool CITY_PAT 50.80.120.2 50.80.120.6 netmask 255.255.255.248
ip nat inside source list 2 pool CITY_PAT overload
ip nat inside source static 172.22.6.254 50.80.120.1 

exit
! write memory == copy running-config startup-config
w m
```


## GLEBE_RT

```
! enable
en
! write erase == erase startup-config
w e
sdm prefer dual-ipv4-and-ipv6 default
! reload
rel

en
conf t
! hostname
h GLEBE
! ipv6 unicast-routing
ipv u
! no ip domain-lookup
no ip domain-l

enable secret class

in g0/0
des Link to CITY
ip address 172.22.14.2 255.255.255.252
ipv6 address 2001:50:80:10D::1/64
no sh

in g0/1.10
description VLAN 10 gateway
! encapsulation dot1Q 10
en d 10
ip addr 172.22.10.1 255.255.255.224
ipv6 addr 2001:50:80:106::/64
ip nat inside

int g0/1.20
description VLAN 20 gateway
en d 20
ip addr 172.22.8.1 255.255.255.0
ipv6 addr 2001:50:80:104::/64
ip helper-address 172.22.14.6
ip nat inside

int g0/1.30
description VLAN 30 gateway
en d 30
ip addr 172.22.9.1 255.255.255.0
ipv6 addr 2001:50:80:105::/64
ip helper-address 172.22.14.6
ip nat inside

int g0/1.137
description Native VLAN 137 gateway
! encapsulation dot1Q 137 native
en d 137 n
ip ad 172.22.10.33 255.255.255.240
ipv6 addr 2001:50:80:107::/64

in g0/1
no shut

int s0/0/1
description Link to ISP
ip addr 50.80.120.22 255.255.255.252
ipv ad 2001:50:80:121::1/64
ip nat ouside
no sh

int Vlan1
sh

router rip
version 2
r s
passive-interface g0/1
passive-interface s0/0/1
network 172.22.0.0
no auto-summary

ip route 0.0.0.0 0.0.0.0 s0/0/1 121
ipv6 route ::/0 s0/0/1 2001:50:80:121:: 5
ipv6 route 2001:50:80:100::/62 GigabitEthernet0/0 2001:50:80:10D
ipv6 route 2001:50:80:108::/62 GigabitEthernet0/0 2001:50:80:10D
ipv6 route 2001:50:80:10C::/64 GigabitEthernet0/0 2001:50:80:10D
! should be only the link between city and chats, so use /64
! ipv6 route 2001:50:80:10C::/62 GigabitEthernet0/0 2001:50:80:10C::1

banner motd #Authorized access only!#


access-list 1 permit host 172.22.6.254
access-list 1 deny any
access-list 2 permit 172.22.6.0 0.0.0.255
access-list 2 permit 172.22.0.0 0.0.3.255
access-list 2 permit 172.22.4.0 0.0.1.255
access-list 2 permit 172.22.10.0 0.0.0.31
access-list 2 permit 172.22.8.0 0.0.0.255
access-list 2 permit 172.22.9.0 0.0.0.255
access-list 2 permit 172.22.13.0 0.0.0.31
access-list 2 permit 172.22.12.0 0.0.0.127
access-list 2 permit 172.22.12.128 0.0.0.127
access-list 2 deny any

ip domain-name ccna-lab.com
! in lab, use crypto key generate rsa modulus 1024
cr k g r
1024
username CaseStudy privilege 15 secret cisco1

line vty 0 4
access-class 1 in
exec-timeout 10
transport input ssh
login local 
logging synchronous
end
show ip ssh

conf t
! ip ssh time-out 75
ip s t 
! ip ssh authentication-retries 2
ip s a 2

conf t
ip nat pool GLEBE_PAT 50.80.120.10 50.80.120.14 netmask 255.255.255.248
ip nat inside source list 2 pool GLEBE_PAT overload
ip nat inside source static 172.22.6.254 50.80.120.9 

conf t
line con 0
exec-timeout 10
password cisco
login
logging synchronous
exit
```


## CHATS_RT

```
! write erase
w e
prefer dual-ipv4-and-ipv6 default
rel

conf t
h CHATS
ipv u
no ip domain-l

enable secret class

in g0/0
no shut

int g0/0.10
description VLAN 10 gateway
! encapsulation dot1Q 10
en d 10
ip addr 172.22.13.1 255.255.255.224
ipv6 addr 2001:50:80:10A::/64

int g0/0.20
description VLAN 20 gateway
! encapsulation dot1Q 20
en d 20
ip addr 172.22.12.1 255.255.255.0
ipv6 addr 2001:50:80:108::/64

int g0/0.30
description VLAN 30 gateway
! encapsulation dot1Q 30
en d 30
ip addr 172.22.12.129 255.255.255.128
ipv6 addr 2001:50:80:109::/64

int g0/0.137
description Native VLAN 137 gateway
! encapsulation dot1Q 137 native
en d 137 n
ip addr 172.22.13.33 255.255.255.240
ipv6 addr 2001:50:80:10B::/64

int g0/0
no shut

int s0/0/1
description Link to CITY
ip addr 172.22.14.6 255.255.255.252
ipv6 addr 2001:50:80:10C::1/64
no shut

int Vlan1
sh

ip route 0.0.0.0 0.0.0.0 s0/0/1 
ipv6 route ::/0 s0/0/1 2001:50:80:10C::

access-list 1 permit host 172.22.6.254
access-list 1 deny any

banner motd #Authorized access only!#

ip dhcp excluded-address 172.22.0.1 172.22.0.5
ip dhcp excluded-address 172.22.4.1 172.22.4.5
ip dhcp excluded-address 172.22.8.1 172.22.8.5
ip dhcp excluded-address 172.22.9.1 172.22.9.5
ip dhcp excluded-address 172.22.12.1 172.22.12.5
ip dhcp excluded-address 172.22.12.129 172.22.12.133

ip dhcp pool CITY_SALES
network 172.22.0.0 255.255.252.0
default-router 172.22.0.1
ip dhcp pool CITY_FINANCE
network 172.22.4.0 255.255.254.0
default-router 172.22.4.1
ip dhcp pool GLEBE_SALES
network 172.22.8.0 255.255.255.0
default-router 172.22.8.1
ip dhcp pool GLEBE_FINANCE
network 172.22.9.0 255.255.255.0
default-router 172.22.9.1
ip dhcp pool CHATS_SALES
network 172.22.12.0 255.255.255.128
default-router 172.22.12.1
ip dhcp pool CHATS_FINANCE
network 172.22.12.128 255.255.255.128
default-router 172.22.12.129

ip domain-name ccna-lab.com
! in lab, use crypto key generate rsa modulus 1024
cr k g r
1024
username CaseStudy privilege 15 secret cisco1

line vty 0 4
access-class 1 in
exec-timeout 10
transport input ssh
login local 
logging synchronous
end
show ip ssh

conf t
! ip ssh time-out 75
ip s t 
! ip ssh authentication-retries 2
ip s a 2

line con 0
exec-timeout 10
password cisco
login
logging synchronous
exit
```

## CITY1_SW

```
! write erase
w e
prefer dual-ipv4-and-ipv6 default
rel

conf t
h CITY1_SW
no ip domain-l

enable secret class

vlan 10
name Excutive
vlan 137
name Management
vlan 459
name Blackhole

! 1 for vlan10(network admin), 1 for vlan137(Trunk), 2 for connections to rt and sw, others for blackhole
int fa0/1
description VLAN 10 interface
switchport mode access
switchport access vlan 10
switchport port-security
! no ip address
no shut

int range fa0/2 - 22
switchport mode access
switchport access vlan 459
switchport port-security
no ip address
no shut

int range f0/23 - 24
des Trunk
switchport trunk native vlan 137
switchport trunk allowed vlan 10,20,30,137
switchport mode trunk
no shut

no vlan 459

int Vlan1
no ip address
shutdown

int Vlan 137
ip addr 172.22.7.1 255.255.255.240
ip default-gateway 172.22.7.17

access-list 1 permit host 172.22.6.254
access-list 1 deny any

banner motd #Authorized access only!#

ip domain-name ccna-lab.com
! in lab, use crypto key generate rsa modulus 1024
cr k g r
1024
username CaseStudy privilege 15 secret cisco1

line vty 0 15
access-class 1 in
exec-timeout 10
transport input ssh
login local 
logging synchronous
end
show ip ssh

conf t
ip ssh time-out 75
ip ssh authentication-retries 2

line con 0
exec-timeout 10
password cisco
login
logging synchronous
exit
```

## CITY2_SW

```
! write erase
w e
prefer dual-ipv4-and-ipv6 default
rel

conf t
h CITY2_SW
no ip domain-l

enable secret class

vlan 10
name Excutive
vlan 20
name Sales
vlan 30
name Finance
vlan 137
name Management
vlan 459
name Blackhole
ex

! vlan10:vlan20:vlan30 = 1:4:2, 1 for vlan137(Trunk), others for blackhole
! in range
in r fa0/1 - 3
description VLAN 10 interface
switchport mode access
switchport access vlan 10
switchport port-security
no shut

in r fa0/4 - 15
description VLAN 20 interface
switchport mode access
switchport access vlan 20
switchport port-security
no shut

in range f0/16 - 21
description VLAN 30 interface
switchport mode access
switchport access vlan 30
switchport port-security
no ip address
no shut

in fa0/24
description Trunk
switchport trunk native vlan 137
switchport trunk allowed vlan 10,20,30,137
switchport mode trunk
no shut

in range fa0/22 - 23
switchport mode access
switchport access vlan 459
switchport port-security
no ip address
no shut

no vlan 459

int Vlan1
no ip address
shutdown

int Vlan 137
ip addr 172.22.7.2 255.255.255.240
ip default-gateway 172.22.7.17

access-list 1 permit host 172.22.6.254
access-list 1 deny any

banner motd #Authorized access only!#

! ip domain-name CITY2_SW.ccna-lab.com
! in lab, use crypto key generate rsa modulus 1024
cr k g r
y
1024
username CaseStudy privilege 15 secret cisco1

line vty 0 15
access-class 1 in
exec-timeout 10
transport input ssh
login local 
logging synchronous
end
show ip ssh

conf t
ip ssh time-out 75
ip ssh authentication-retries 2

line con 0
exec-timeout 10
password cisco
login
logging synchronous
exit
```

## GLEBE_SW

```
! write erase
w e
prefer dual-ipv4-and-ipv6 default
rel

conf t
h GLEBE_SW
no ip domain-l

enable secret class

vlan 10
name Excutive
vlan 20
name Sales
vlan 30
name Finance
vlan 137
name Management
vlan 459
name Blackhole
ex

# vlan10:vlan20:vlan30 = 1:8.5:8.5, 1 for vlan137(Trunk), others for blackhole
int fa0/1
description VLAN 10 interface
switchport mode access
switchport access vlan 10
switchport port-security
no sh

int r fa0/2 - 10
description VLAN 20 interface
switchport mode access
switchport access vlan 20
switchport port-security
no ip address
no sh

int range fa0/11 - 19
description VLAN 30 interface
switchport mode access
switchport access vlan 30
switchport port-security
no ip address
no sh

int range fa0/20 - 23
switchport mode access
switchport access vlan 459
switchport port-security
no ip address
no shut

int fa0/24
description Trunk
switchport trunk native vlan 137
switchport trunk allowed vlan 10,20,30,137
switchport mode trunk
no shut

no vlan 459

! check whether have others unused interfaces, if so, put them in blackhole vlan.


int Vlan 137
ip addr 172.22.10.33 255.255.255.240
ip default-gateway 172.22.10.33

int Vlan1
no ip address
shutdown

access-list 1 permit host 172.22.6.254
access-list 1 deny any

banner motd #Authorized access only!#

ip domain-name ccna-lab.com
crypto key generate rsa modulus 1024
username CaseStudy privilege 15 secret cisco1

line vty 0 15
access-class 1 in
exec-timeout 10
transport input ssh
login local 
logging synchronous
end
show ip ssh

conf t
ip ssh time-out 75
ip ssh authentication-retries 2

line con 0
exec-timeout 10
password cisco
login
logging synchronous
exit
```

## CHATS_SW

```
! write erase
w e
prefer dual-ipv4-and-ipv6 default
rel

conf t
h GLEBE_SW
no ip domain-l
ipv u

enable secret class

vlan 10
name Excutive
vlan 20
name Sales
vlan 30
name Finance
vlan 137
name Management
vlan 459
name Blackhole
ex

! vlan10:vlan20:vlan30 = 1:4:4, 1 for vlan137(Trunk), others for blackhole

int r fa0/1 - 2
description VLAN 10 interface
switchport mode access
switchport access vlan 10
switchport port-security
no shut

int r fa0/3 - 10
description VLAN 20 interface
switchport mode access
switchport access vlan 20
switchport port-security
no ip address
no shut

int range fa0/11 - 19
description VLAN 30 interface
switchport mode access
switchport access vlan 30
switchport port-security
no ip address
no shut

int range fa0/20 - 23
switchport mode access
switchport access vlan 459
switchport port-security
no ip address
no shut

int fa0/24
description Trunk
switchport trunk native vlan 137
switchport trunk allowed vlan 10,20,30,137
switchport mode trunk
no shut

no vlan 459

int Vlan 137
ip addr 172.22.13.34 255.255.255.240
ip default-gateway 172.22.13.33

int Vlan1
no ip address
shutdown

access-list 1 permit host 172.22.6.254
access-list 1 deny any

banner motd #Authorized access only!#

ip domain-name ccna-lab.com
! crypto key generate rsa modulus 1024
cr k g r
1024

username CaseStudy privilege 15 secret cisco1

line vty 0 15
access-class 1 in
exec-timeout 10
transport input ssh
login local 
logging synchronous
end
show ip ssh

conf t
ip ssh time-out 75
ip ssh authentication-retries 2

line con 0
exec-timeout 10
password cisco
login
logging synchronous
exit
```

## Network Admin(vlan 10)

```
# link to CITY_RT GE0/1.10
ip address: 172.22.6.254
default gateway: 172.22.6.1
subnet mask 255.255.255.0
ipv6 address: 2001:50:80:102:FFFF:FFFF:FFFF:FFFF
ipv6 default gateway: 2001:50:80:102::
```

## GLEBE PC(vlan 20)

```
# link to GLEBE_RT GE0/1.20
ip address: 172.22.8.2
default gateway: 172.22.8.1
subnet mask 255.255.255.0
ipv6 address: 2001:50:80:104::1/64
ipv6 default gateway: 2001:50:80:104::
```

## CHATS PC(vlan 10)

```
# link to CHAT_RT GE0/1.10
ip address: 172.22.13.2
default gateway: 172.22.13.1
subnet mask 255.255.255.224
ipv6 address: 2001:50:80:10A::1/64
ipv6 default gateway: 2001:50:80:10A::
```
## Password Revovery
reboot
open Putty and hold `break` / `Ctr-break`
confreg 0x2142
reset
copy s r
config-register 0x2102
c r s

## Troubleshooting Commands
```
sh ip i b
sh ip ro
sh ipv ro
! show ip ssh
sh ip s
sh protocols
show port-security
show access-lists 2
```

## Connection Order
123
456
10, 11, 12

GLEBE_R con
CHATS_R con
GLEBE_SW con
CHATS_SW con
ISP_R con
CITY_R con
CITY_SW con
CITY2_SW con
GLEBE - CHATS
CITY - GLEBE
CITY - CHATS

## Test Case
normal: all hosts ping 11.11.11.11 & 2001:11:11:11:：11
failure: all hosts ping 11.11.11.11 & 2001:11:11:11:：11
host ssh to devices: should be denied
NA ssh to devices
DHCP, SLAAC
