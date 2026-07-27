# srv-deb-01 — Serveur Debian 13

## Rôle dans l'infrastructure

srv-deb-01 est le serveur applicatif de DeltaWay. Contrairement à
srv-win-01 qui héberge les services d'infrastructure (AD, DNS, DHCP),
ce serveur est dédié aux services métier et applicatifs qui évolueront
au fil des phases du projet.

## Configuration réseau

| Paramètre | Valeur |
|-----------|--------|
| IP | 192.168.10.20/24 |
| Passerelle | 192.168.10.254 (pfSense) |
| DNS primaire | 192.168.10.10 (srv-win-01) |
| DNS secondaire | 9.9.9.9 (Quad9 — secours) |

Configuration dans `/etc/network/interfaces` :

```bash
auto ens33
iface ens33 inet static
    address 192.168.10.20
    netmask 255.255.255.0
    gateway 192.168.10.254
    dns-nameservers 192.168.10.10 9.9.9.9
```

## Installation

Debian 13 installée en mode texte sans interface graphique — c'est
un serveur, pas un poste de travail. Le partitionnement a été fait
avec LVM pour permettre un redimensionnement des partitions à chaud
si nécessaire, avec séparation de /var et /srv — bonne pratique en
production pour éviter qu'un remplissage de logs impacte le système.

L'installation a été faite sans miroir réseau car la VM n'avait pas
accès à internet pendant l'installation (pas de passerelle configurée).
Les dépôts ont été configurés manuellement après installation.

## Configuration des dépôts

Installation sans accès internet — les dépôts ont été configurés
manuellement dans `/etc/apt/sources.list` après installation :

```bash
deb http://ftp.fr.debian.org/debian trixie main
deb http://ftp.fr.debian.org/debian trixie-updates main
deb http://security.debian.org/debian-security trixie-security main
```

Miroir français choisi pour des raisons de proximité géographique
et de conformité RGPD.

## Incident — /etc/resolv.conf inexistant

Après configuration des dépôts, `apt update` échouait avec des
erreurs de résolution DNS. En vérifiant `/etc/resolv.conf` le fichier
n'existait tout simplement pas — je pense qu'il ne s'est pas créé pendant
l'installation car cette dernière a été faite sans configuration réseau et
en mode très minimaliste.

Création manuelle du fichier :

```bash
echo "nameserver 192.168.10.10" > /etc/resolv.conf
echo "nameserver 9.9.9.9" >> /etc/resolv.conf
```

Leçon : sur Debian, `/etc/resolv.conf` n'est pas toujours créé
automatiquement — notamment quand l'installation se fait sans réseau.
Toujours vérifier après installation que la résolution DNS fonctionne.

## SSH

OpenSSH installé pour l'administration à distance :

```bash
apt install openssh-server -y
```

En entreprise on n'administre jamais un serveur physiquement devant
lui — les serveurs sont dans des salles machines ou datacenters.
SSH permet d'administrer le serveur depuis n'importe quel poste
du réseau, d'ouvrir plusieurs sessions simultanées et de scripter
des tâches d'administration.

Accès depuis l'hôte Ubuntu :
```bash
ssh kenny@192.168.10.20
```

## Services à venir

Le serveur est actuellement vide. Il sera enrichi au fil des phases :

| Phase | Services |
|-------|---------|
| 1 — Infrastructure | Apache (intranet) + PHP, MariaDB, GLPI |
| 2 — Sécurisation | Fail2ban, Rsyslog (collecte logs), Keycloak |
| 3 — Cloud | Docker 



