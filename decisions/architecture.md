# Décisions d'architecture — Phase 1

## Adressage et VLANs

pfSense — routeur/firewall central
  → VLAN 10 — Serveurs — 192.168.10.0/24
      passerelle : 192.168.10.1 (pfSense)
      - Windows Server 2022 → 192.168.10.10
      - Debian → 192.168.10.20
  
  → VLAN 20 — Clients — 192.168.20.0/24
      passerelle : 192.168.20.1 (pfSense)
      - Windows 11 → 192.168.20.10

Pourquoi 192.168.x.x → adresses privées les plus utilisées (non routable sur
internet)
Pourquoi /24 → 254 machines, simple à gérer
Pourquoi VLANs → segmentation, isolation si incident
pfSense a plusieurs IP → une par réseau qu'il connecte

## Convention de nommage

Convention nommage : rôle-os-numéro pour bien identifier la machine (un coup 
d'oeil dans les logs ou supervision et on sait ce que fait chaque machine)

Routeur/Firewall : fw-pfsense-01
Windows Server : srv-win-01
Debian Server : srv-deb-01
Windows client : pc-win-01

## Convention IPs/DHCP

On commence à 192.168.10.10 pour le vlan serveurs car on garde les IPs en dessous
pour d'autres équipements réseaux et infra si besoin (192.168.10.2-192.168.10.9).
Ensuite on garde 10 IPs (192.168.10.10-.19) pour les serveurs Windows puis 10 ips 
(192.168.10.20-.29) pour les serveurs Debian puis 10 IPs (.30-.39) pour les serveurs 
applicatifs (serveur qui héberge seulement une ou des applications métiers =/= serveurs 
infra qui sont indispensables pour que le réseau fonctionne). 

Pour les clients ce sera en dynamique dhcp (pool .100-.254). 
On garde .1 pour le pfsense puis .2-.9 pour tout autre équipement réseau
On garde .10-.99 pour des IPs fixes ou réservations dhcp si besoin (imprimantes, caméras, 
téléphones VOIP etc). Si on a besoin de plus d'adresses on peut réduire le pool dhcp.
On met les clients en dhcp en entreprise il peut y avoir 200 machines donc on ne va pas s'amuser 
à mettre des IPs fixes sur 200 machines (erreurs, maintenance, trop long). Un serveur doit être 
trouvable à chaque fois donc IP fixe =/= des clients qui n'ont pas besoin d'être trouvé par leur IP. 
Libérer les anciennes IPs aussi et en attribuer de nouvelles.

Pour commencer on va mettre le dhcp sur le pfsense qui a un dhcp intégré et ce sera la première vm de 
créée ce qui permettra au client d'avoir tout de suite une IP mais plus tard on mettra le dhcp sur le 
Windows Server lorsqu'on créera un AD pour profiter de l'intégration AD/DNS.



