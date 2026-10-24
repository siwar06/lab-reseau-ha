# Lab réseau d'entreprise haute disponibilité (EVE-NG)

Réseau d'entreprise redondant : accès par VLANs, deux switchs cœur en HSRP (load-sharing) avec DHCP en split-scope, EtherChannel LACP vers deux routeurs de bordure, OSPF jusqu'aux branches, pare-feu pfSense en haute disponibilité (CARP) avec NAT vers Internet, et une DMZ isolée.

## Ordre de lecture (preuves)

| # | Contenu | Dossier |
|---|---|---|
| 1 | Topologie | [docs/01-topologie.md](docs/01-topologie.md) |
| 2 | Sorties Cisco : `show standby brief`, `show etherchannel summary`, `show ip ospf neighbor` | [docs/02-verifications-cisco.md](docs/02-verifications-cisco.md) |
| 3 | Captures pfSense : CARP, règles, NAT | [docs/03-pfsense.md](docs/03-pfsense.md) |
| 4 | Tests ping et DMZ | [docs/04-tests.md](docs/04-tests.md) |
| 5 | Problèmes rencontrés et solutions | [docs/05-problemes-rencontres.md](docs/05-problemes-rencontres.md) |
| - | Configurations | [configs/](configs/) |

## Architecture

| Équipement | Rôle |
|---|---|
| Sw-core1, Sw-core2 | Cœur L3 : routage inter-VLAN, HSRP load-sharing, DHCP split-scope, OSPF |
| 4 switchs d'accès | VLANs 10, 20, 30 (trunks 802.1Q vers les 2 cœurs) |
| L3-edge1, L3-edge2 | Bordure : OSPF + liens série vers les branches |
| Branche1, Branche2 | Sites distants |
| pfSense1, pfSense2 | Pare-feu HA (CARP + pfsync + XMLRPC), NAT |
| R20 | Switch de la DMZ |

## Plan d'adressage

| VLAN | Nom | Réseau | Passerelle (VIP) |
|---|---|---|---|
| 10 | Étudiants | 10.1.10.0/24 | 10.1.10.1 (HSRP) |
| 20 | Enseignants | 10.1.20.0/24 | 10.1.20.1 (HSRP) |
| 30 | Administration | 10.1.30.0/24 | 10.1.30.1 (HSRP) |
| 80 | Transit cœur ↔ pfSense | 10.1.80.0/24 | 10.1.80.1 (HSRP) · 10.1.80.4 (CARP pfSense) |
| 100 | DMZ | 10.1.100.0/24 | 10.1.100.1 (CARP) |
| 40/50/60/70 | Liens cœur ↔ edge | 40.0.0.0/8 … 70.0.0.0/8 | - |

## Technologies

HSRP · DHCP split-scope · 802.1Q · EtherChannel LACP · OSPF area 0 · CARP · pfsync · NAT (PAT) · DMZ · EVE-NG (IOL) · pfSense Plus

## Sécurité du dépôt

Aucun mot de passe, `enable secret`, hash ou clé n'est publié. Les exports `config.xml` de pfSense sont exclus (`.gitignore`) : on utilise des captures d'écran.
