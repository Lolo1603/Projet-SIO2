# Configuration du switch



Réinitialisation du switch `erase config-startup`


<br>
## Renommage du switch
 
`enable`

`configure terminal`

`hostname ‘nom du switch’`

<br>
## Créer les différents Vlan nécessaires

`vlan [numéro du port]` puis `name [nom_du_VLAN]`

<br>
## Attribuer des ports aux VLANS

`interface [numéro du port]`

`switchport mode access/trunk`

`Switchport access vlan [numéro de vlan]`

<br>
## Creation d'un utilisateur

`username [admin] privilege 15 secret [password]`

<br>
## Activation du ssh

Et nous activons les interfaces avec la commande 
`ip domain name cha.chartres.sportludique.local`

`ip ssh version 2`

`crypto key generate-keys modulus 1024`

`line vty 0 4`

`login local`

`transport input ssh`

`write memory`

<br>
<br>
# Configuration Routeur

## Mise en place du NAT

Interface du VLAN 120 `interface gig0/0.120` avec l'adresse ip `10.28.2.253 255.255.255.0` 

Interface du VLAN 222 `ip address 192.168.222.254 255.255.255.0` et `ip nat inside` avec `l'encapsulation dot1Q`

Interface WAN `ip address 221.87.128.1 255.255.255.252` et `ip nat outside`

`no shutdown`

Route par défaut : `ip route 0.0.0.0 0.0.0.0 221.87.128.2`

On mets une acess-list pour avoir internet sur tous les réseaux sauf Mana `access-list 1 permit 172.28.160.0 0.0.31.255`

##Mise en place internet sur les vlans

On mets la route par défaut pour le switch : `0.0.0.0 0.0.0.0 192.168.222.254`

On mets la route sur le routeur pour qu'il connaisse le VLAN 221: `172.28.161.0 255.255.255.0 192.168.222.1`