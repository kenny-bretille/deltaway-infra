# Décisions d'architecture — Phase 1

## Adressage réseau

J'ai choisi la plage 192.168.x.x car ce sont des adresses privées
RFC 1918, non routables sur internet et réservées aux réseaux locaux.
Le masque /24 offre 254 adresses utilisables, suffisant pour DeltaWay.

## Segmentation réseau

L'infrastructure est divisée en deux VLANs pour isoler les serveurs
des postes clients. En cas d'incident sur un poste client, les serveurs
restent protégés et inaccessibles directement.

## Convention de nommage

Format retenu : rôle-os-numéro (ex: srv-win-01).
Permet d'identifier immédiatement le rôle d'une machine dans les logs
ou un outil de supervision sans avoir à chercher.

## Plan d'adressage

| Machine | IP | Rôle |
|---------|-----|------|
| fw-pfsense-01 | 192.168.10.1 / 192.168.20.1 | Routeur/Firewall |
| srv-win-01 | 192.168.10.10 | Windows Server 2022 — AD |
| srv-deb-01 | 192.168.10.20 | Debian 13 — Services |
| pc-win-01 | DHCP 192.168.20.100-254 | Poste client Windows 11 |

## DHCP

Les postes clients obtiennent leur IP dynamiquement via DHCP —
en entreprise gérer des IPs fixes sur des centaines de postes
serait une source d'erreurs et de maintenance inutile.

Le DHCP est provisoirement sur pfSense car c'est la première VM
déployée. Il sera migré sur Windows Server une fois l'AD en place
pour profiter de l'intégration AD/DNS.

## Convention d'adressage

Pour chaque réseau :
- `.1` → pfSense (passerelle)
- `.2` à `.9` → réservés équipements réseau
- `.10` à `.99` → IPs fixes et réservations DHCP (imprimantes, caméras, téléphones VoIP)
- `.100` à `.254` → pool DHCP postes clients

Sur le VLAN serveurs on regroupe par type :
- `.10` à `.19` → serveurs Windows
- `.20` à `.29` → serveurs Debian
- `.30` à `.39` → serveurs applicatifs

Cette convention permet d'identifier le rôle d'une machine
rien qu'à son adresse IP.
