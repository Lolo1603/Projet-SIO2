# Configuration du switch



Réinitialisation du switch `erase config-startup`


<br>

## Renommage du switch
```
 enable
 configure terminal
 hostname ‘nom du switch'
```
<br>
## Créer les différents Vlan nécessaires

`vlan [numéro du port]` puis `name [nom_du_VLAN]`

<br>

## Attribuer des ports aux VLANS
```
interface [numéro du port]
switchport mode access/trunk
Switchport access vlan [numéro de vlan]
```

<br>

## Creation d'un utilisateur

`username [admin] privilege 15 secret [password]`

<br>

## Activation du ssh

Et nous activons les interfaces avec la commande 
```
ip domain name cha.chartres.sportludique.local
ip ssh version 2
crypto key generate-keys modulus 1024
line vty 0 4
login local
transport input ssh
write memory
```
<br>
<br>

# Configuration Routeur

<br>

## Mise en place du NAT

Interface du VLAN 120 `interface gig0/0.120` avec l'adresse ip `10.28.2.253 255.255.255.0` 

Interface du VLAN 222 `ip address 192.168.222.254 255.255.255.0` et `ip nat inside` avec `l'encapsulation dot1Q`

Interface WAN `ip address 221.87.128.1 255.255.255.252` et `ip nat outside`

`no shutdown`

Route par défaut : `ip route 0.0.0.0 0.0.0.0 221.87.128.2`

On mets une acess-list pour avoir internet sur tous les réseaux sauf Mana `access-list 1 permit 172.28.160.0 0.0.31.255`

<br>

## Mise en place internet sur les vlans

On mets la route par défaut pour le switch : `0.0.0.0 0.0.0.0 192.168.222.254`

On mets la route sur le routeur pour qu'il connaisse le VLAN 221: `172.28.161.0 255.255.255.0 192.168.222.1`

<br>

## Mise en place du 2eme Routeur

On met un deuxieme routeur pour assurer la disponibilité

On copie la conf du premier routeur sur le deuxieme

On config HSRP sur les 2 routeurs

R1:
```
 standby 1 ip 192.168.222.253
 standby 1 priority 110
 standby 1 preempt
```

R2:
```
 standby 1 ip 192.168.222.252
 standby 1 priority 100
 standby 1 preempt
```

# Configuration de firewall

<br>

## Mise a jour du firewall

Mettre une date en 2025

Redemarrer le firewall

Installer le fichier de mise a jour

Mettre le firewall a l'heure

<br>

## Mise en place des sous interface

Nous avons mis les vlan mana et lan en sous interface dans l'interface in en 802.1Q

## Mise en place des routes

Nous avons mis la route du reseau serveur pour qu'il puisse communiquer avec le firewall
```
Dest : Serveur
Interface : lan
Plan adressage : 172.28.160.0/24
Passerelle : Pass-Serveur (192.168.223.1)
```
## Filtre et NAT

Nous avons mis le firewall en pass all pour tester si tout fonctionne

## Modif switch

On a rajouter les vlan 223 et 224

Le port WAN et la DMZ sont en mode access

Le port lan est en trunk pour faire passer le vlan Mana et Lan

On a modifier la route par défaut

<br>
<br>

# Mise en place des serveurs

<br>

## Installation du DC1 & GUI1

Le DC1 sera en mode core et le gui sera une interface graphique.

Pour la création des VMs les ressources sont les mêmes : 2 coeurs, 4go de RAM.

Nous les avons mis sur deux noeuds différents pour éviter la surcharge sur un seule noeud.


CHA-DC1 : 172.28.160.1/24
CHA-GUI1 : 172.28.160.2/24

On renomme les deux serveurs et on configure l'adressage réseau avant d'installer le rôle AD sur le CHA-DC1.

Installation du rôle AD : 
```
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName "cha.chartres.sportludique.fr" -DomainNetbiosName "CHA" -InstallDNS
```
Nous ajoutons un serveur local à gérer sur le serveur CHA-GUI1 et on ajoute CHA-DC1 à gérer.

Des instatanées doivent être pris régulièrement pour nous permettre un retour en arrière.


## Configuration de GUI1

Nous avons mis le GUI1 dans le VLAN serveur puis dans le domaine 

On a rajouter une interface dans le vlan MANA pour l'administrer depuis le mana

### Probleme

Interface MANA n'arriver pas a ping la passerelle

On a vus que sur le switch le port qui est relier au huawei sur le vlan MANA était broken

Après 2 heure de recherche le prof a regler le probleme en retirent le spamming-tree sur le vlan MANA

Le probleme peut etre lié au Huawei