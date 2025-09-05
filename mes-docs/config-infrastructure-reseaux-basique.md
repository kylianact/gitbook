# Config infrastructure réseaux basique

Routeur (Cisco IOS)

```python
! ========= Interfaces =========
! Vers Internet
interface GigabitEthernet0/3/0
description Vers FAI (Internet)
ip address dhcp
ip nat outside
no shutdown
! Vers DMZ
interface FastEthernet0/1
description DMZ
ip address 192.168.50.254 255.255.255.0
ip nat inside
no shutdown
! Trunk vers le cœur (routage inter-VLAN)
interface FastEthernet0/0
description Trunk vers switch de coeur
no shutdown
! === Sous-interfaces (IMPERATIF: f0/0.) ===
interface FastEthernet0/0.10
encapsulation dot1Q 10
ip address 192.168.10.254 255.255.255.0
ip nat inside
interface FastEthernet0/0.20
encapsulation dot1Q 20
ip address 192.168.20.254 255.255.255.0
ip nat inside
interface FastEthernet0/0.100
encapsulation dot1Q 100
ip address 192.168.100.254 255.255.255.0
ip nat inside
interface FastEthernet0/0.222
encapsulation dot1Q 222
ip address 192.168.222.254 255.255.255.0
ip nat inside
```

### NAT / PAT

(D’abord la sortie Internet pour tous les VLAN privés.)

```python
! Tout le privé 192.168.0.0/16 vers Internet via l’IP WAN
access-list 100 permit ip 192.168.0.0 0.0.255.255 any
ip nat inside source list 100 interface GigabitEthernet0/3/0 overload

```

### DNAT (publication DMZ)

(Publie le Web et le DNS de la DMZ sur l’IP publique 1.2.3.4 comme demandé.)

```python
! Exemples d’IP DMZ :
!  - Web DMZ  : 192.168.50.10 (HTTP/HTTPS)
!  - DNS DMZ  : 192.168.50.53 (UDP/TCP 53)

ip nat inside source static tcp 192.168.50.10 80  1.2.3.4 80
ip nat inside source static tcp 192.168.50.10 443 1.2.3.4 443
ip nat inside source static udp 192.168.50.53 53  1.2.3.4 53
ip nat inside source static tcp 192.168.50.53 53  1.2.3.4 53
```

### Route par défaut vers le FAI

(au choix selon ta maquette Packet Tracer)

```python
! Option 1 (next-hop connu)
ip route 0.0.0.0 0.0.0.0 <IP_next-hop_FAI>

! Option 2 (par l’interface WAN)
! ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/3/0
```

### ACL “serveurs” (sur VLAN 100)

* DGI (VLAN 20) : accès **sans restriction** aux serveurs (192.168.100.0/24)
* Commerciaux (VLAN 10) : **seulement** les ports/protocoles utiles
* Tout le reste : **interdit**\
  À appliquer **en sortie** de `Fa0/0.100` (côté routeur vers VLAN 100), comme préconisé.

```python
ip access-list extended ACL_V100_OUT
 remark DGI -> serveurs : full
 permit ip 192.168.20.0 0.0.0.255 192.168.100.0 0.0.0.255

 remark COMMERCIAL -> web interne (exemple)
 permit tcp 192.168.10.0 0.0.0.255 192.168.100.0 0.0.0.255 eq 80
 permit tcp 192.168.10.0 0.0.0.255 192.168.100.0 0.0.0.255 eq 443
 ! (ajoute d’autres lignes si tu as d’autres services autorisés)

 remark Tout autre trafic vers 192.168.100.0/24 : interdit
 deny   ip any 192.168.100.0 0.0.0.255

 remark Laisse passer le reste (autres destinations)
 permit ip any any

interface FastEthernet0/0.100
 ip access-group ACL_V100_OUT out
```

### Protection SSH sur le routeur (VTY)

(Limiter l’entrée VTY aux machines du VLAN DGI.)

```python
! Pré-requis SSH
hostname Boss
ip domain-name cyb509.rt
crypto key generate rsa modulus 1024
username admin privilege 15 secret Admin@123

line vty 0 4
 login local
 transport input ssh

! Filtre d’entrée VTY (seules les IP 192.168.20.0/24)
access-list 20 permit 192.168.20.0 0.0.0.255
line vty 0 4
 access-class 20 in
```

