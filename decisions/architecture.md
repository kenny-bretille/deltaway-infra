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

## Mise à jour — Changement adressage passerelles pfSense

**Date :** Août 2026

### Le problème

Les passerelles pfSense étaient initialement configurées en `.1` :
- Interface SERVEURS : `192.168.10.1`
- Interface CLIENTS : `192.168.20.1`

VMware Workstation étant un hyperviseur de **type 2** il s'installe
sur un OS hôte — ici Ubuntu. VMware attribue automatiquement l'adresse
`.1` de chaque subnet à l'hôte Ubuntu sur ses interfaces virtuelles
vmnet2 et vmnet3. Cela créait un conflit d'adresse permanent avec les
passerelles pfSense au redémarrage.

Plusieurs tentatives de correction ont été faites — configuration
Netplan, service systemd — sans résultat durable car VMware réinitialise
ses interfaces après chaque démarrage, écrasant toute configuration
appliquée par l'OS hôte.

### Ce problème n'existe pas en production

Avec un hyperviseur de **type 1** comme ESXi, Hyper-V ou Proxmox il
n'y a pas d'OS hôte en dessous — l'hyperviseur est directement sur
le matériel. Il n'y a donc aucune interface hôte qui prend une IP
sur les réseaux virtuels. C'est un avantage concret du type 1 en
production.

### Solution retenue

Passage des passerelles pfSense en `.254` :
- Interface SERVEURS : `192.168.10.254`
- Interface CLIENTS : `192.168.20.254`

Cette convention évite définitivement tout conflit avec VMware qui
garde ses `.1` automatiques.

### Convention d'adressage finale

| Machine | IP |
|---------|-----|
| fw-pfsense-01 SERVEURS | 192.168.10.254 |
| fw-pfsense-01 CLIENTS | 192.168.20.254 |
| srv-win-01 | 192.168.10.10 |
| srv-deb-01 | 192.168.10.20 |
| pc-win-01 | DHCP 192.168.20.100-253 |

