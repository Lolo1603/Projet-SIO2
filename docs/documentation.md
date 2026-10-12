# <font color= 'red'> 1 - Configuration du switch </font>


Réinitialisation du switch `erase config-startup`

## 1.1) Renommage du switch
```
 enable
 configure terminal
 hostname ‘nom du switch'
```

## 1.2) Créer les différents Vlan nécessaires

`vlan [numéro du port]` puis `name [nom_du_VLAN]`

## 1.3) Attribuer des ports aux VLANS
```
interface [numéro du port]
switchport mode access/trunk
Switchport access vlan [numéro de vlan]
```
## 1.4) Creation d'un utilisateur

`username [admin] privilege 15 secret [password]`

## 1.5) Activation du ssh

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
## 1.6) Modification du switch après la mise en place du Pare-Feu

On a ajouté le VLAN 223.

Le port WAN est en mode access.

Le port LAN est en trunk pour faire passer les VLAN Mana et LAN.

Nous avons donc mis en place la nouvelle route par défaut, qui est la suivante : 0.0.0.0 0.0.0.0 192.168.223.254 ; elle va nous permettre de rediriger tous les paquets vers le pare-feu.

Exemple : je veux sortir sur Internet en étant dans le VLAN Serveurs pour faire des requêtes HTTPS sur Internet.

<br>

# <font color= 'red'> 2 - Configuration Routeur </font>

## 2.1) Mise en place du NAT

Interface du VLAN 120 `interface gig0/0.120` avec l'adresse ip `10.28.2.253 255.255.255.0` 

Interface du VLAN 222 `ip address 192.168.222.254 255.255.255.0` et `ip nat inside` avec `l'encapsulation dot1Q`
Configuration
Interface WAN `ip address 221.87.128.1 255.255.255.252` et `ip nat outside`

`no shutdown`

Route par défaut : `ip route 0.0.0.0 0.0.0.0 221.87.128.2`

On mets une acess-list pour avoir internet sur tous les réseaux sauf Mana `access-list 1 permit 172.28.160.0 0.0.31.255`

## 2.2) Mise en place internet sur les vlans

On mets la route par défaut pour le switch : `0.0.0.0 0.0.0.0 192.168.222.254`

On mets la route sur le routeur pour qu'il connaisse le VLAN 221: `172.28.161.0 255.255.255.0 192.168.222.1`
Configuration
## 2.3) Mise en place du 2eme Routeur

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
<br>

# <font color= 'red'> 3 - Configuration de firewall </font>

## 3.1) Mise a jour du firewall

Mettre une date en 2025.
Redémarrer le firewall.
Installation de la licence.
Installation de la mise à jour via son fichier.
Après toutes ces étapes, définir l'heure du firewall avec le NTP.

## 3.2) Mise en place des sous-interfaces

Nous avons mis en place des interfaces virtuelles sur le notre firewall.
Nous désactivons l'interface « in » (interface physique) pour ensuite mettre en place les interfaces virtuelles, MANA puis LAN.

## 3.3) Mise en place des routes

Nous avons mis en place la route permettant au pare-feu de renvoyer les paquets à tous les VLAN du réseau.
```
Dest : VLAN-INTERNE
Interface : lan
Plan adressage : 172.28.160.0/19
Passerelle : Pass-Serveur (192.168.223.1)
```
## 3.4) Filtre et NAT

Nous avons mis le pare-feu en « pass all » pour finaliser la configuration ; ainsi, nous pourrons réactiver la règle « block all » après la fin de la configuration.

<br>

# <font color= 'red'> 4 - Mise en place des serveurs </font>


## 4.1) Installation du DC1 & GUI1

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

## 4.2)Configuration de GUI1

Nous avons mis le GUI1 dans le VLAN serveur, puis dans le domaine.

On a ajouté une carte réseau pour l'administrer depuis le VLAN Mana.

Le serveur CHA-GUI1 a donc deux cartes réseau, soit une attribuée au VLAN Serveurs pour communiquer avec CHA-DC1 et la deuxième attribuée sur le VLAN Mana pour la manager.

### Probleme

Nous souhaitions accéder à l'interface de management du serveur CHA-GUI1, le switch désactiver directement un port, nous ne pouvions pas accéder au serveur CHA-GUI1 via la VLAN Mana.

Lorsque nous avons établi le raccordement, nous avons constaté sur le switch Cisco que le port se désactivait ; nous avons conclu que le protocole Spanning Tree le désactivait à cause d'une potentielle boucle sur le réseau.

Nous avons désactivé temporairement le protocole Spanning Tree pour nous permettre de connaître l'origine de la boucle ou, à défaut, de voir si une boucle est bien présente.

Il s'est avéré qu'aucune boucle n'était présente sur notre réseau ; nous avons désactivé Spanning Tree et tout fonctionnait correctement.

Après deux longues heures de recherche minutieuse, nous avons trouvé une solution : la mise en place du protocole EtherChannel LACP avec les ports Gig1/0/23 et Gig1/0/24, qui transporteront uniquement les VLAN 120 et 220.

```
conf t

interface range GigabitEthernet1/0/23-24
 description LACP-ETHERCHANNEL
 switchport mode trunk
 switchport trunk allowed vlan 120,220
 channel-group 1 mode active
 no shutdown
 exit

interface Port-channel1
 description LACP-TRUNK
 switchport mode trunk
 switchport trunk allowed vlan 120,220
 no shutdown
 exit

end
write memory
```
Pour ensuite tester que notre solution fonctionne bien, nous avons remis en place le protocole Spanning Tree et nous avons testé l'accès au serveur CHA-GUI1 via son interface de Mana et le test fut un succès.

Nous supposons que l'erreur vient d'une mauvaise réinitialisation du switch.