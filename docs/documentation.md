# Configuration du switch



Réinitialisation du switch `erase config-startup`
Cette commande permet de supprimer la configuration actuelle du switch.

<br>
## Renommage du switch
 
Passer en mode privilégié avec la commande suivante : `enable`

Taper la commande suivante pour entrer dans le mode configuration terminal : `configure terminal`

Pour le renommer, on entrera la commande suivante : `hostname ‘nom du switch’`

<br>
## Créer les différents Vlan nécessaires

Toujours en mode privilégié tapez la commande suivante `vlan [numéro du port]` puis `name [nom_du_VLAN]`

<br>
## Attribuer des ports aux VLANS

Tout en étant dans la configuration du switch, entrez la commande suivante : `interface [numéro du port]`

Définir le mode d’accès : `switchport mode access/trunk`

Associer le port au VLAN : `Switchport access vlan [numéro de vlan]`

<br>
## Creation d'un utilisateur

Entrer dans la config du switch `username [admin] privilege 15 secret [password]`

<br>
## Activation du ssh

Configurer le nom de domaine avec la commande suivante: `ip domain name cha.chartres.sportludique.local`

Activer la version SSH la plus élevée prise en charge avec la commande `ip ssh version 2`

Générer la pîre de clés RSA pour SSH : `crypto key generate-keys modulus 1024`

Passez en mode configuration de lignes VTY avec la commande : `line vty 0 4`

Configurez les lignes vty : `login local`

Limitez l'accès VTY au transport SSH uniquement avec cette commande : `transport input ssh`

On enrengistre la configuration : `write memory`