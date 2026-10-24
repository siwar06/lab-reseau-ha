# 3. Captures pfSense

Prendre les captures sur **pfSense1 (MASTER)** et, quand c'est précisé, sur **pfSense2 (BACKUP)**. Masquer tout mot de passe / clé.

| # | Page pfSense | Fichier | À montrer |
|---|---|---|---|
| 1 | Dashboard > Interfaces | `images/03-interfaces.png` | WAN, LAN (10.1.80.2), SYNC (200.200.200.1), DMZ (10.1.100.253) en *up* |
| 2 | Status > CARP (pfSense1) | `images/03-carp-pfsense1.png` | LAN@80 `10.1.80.4` et DMZ@100 `10.1.100.1` en **MASTER** |
| 3 | Status > CARP (pfSense2) | `images/03-carp-pfsense2.png` | les mêmes VIP en **BACKUP** |
| 4 | Firewall > Virtual IPs | `images/03-virtual-ips.png` | VIP CARP, VHID 80 et 100 |
| 5 | System > Routing > Gateways | `images/03-gateways.png` | `CORE_HSRP` = 10.1.80.1 |
| 6 | System > Routing > Static Routes | `images/03-static-routes.png` | 10.1.10/20/30.0/24 et les 2 branches via `CORE_HSRP` |
| 7 | Firewall > Aliases | `images/03-alias.png` | `INTERNAL_NETS` |
| 8 | Firewall > Rules > LAN | `images/03-rules-lan.png` | règle `INTERNAL_NETS` → any |
| 9 | Firewall > Rules > DMZ | `images/03-rules-dmz.png` | ordre : ICMP vers This Firewall, Block `INTERNAL_NETS`, Pass any |
| 10 | Firewall > NAT > Outbound | `images/03-nat-outbound.png` | mode **Hybrid**, règles WAN pour 10.1.0.0/16, branches, 10.1.100.0/24 |
| 11 | Status > CARP (section State Synchronization) | `images/03-pfsync.png` | pfsync actif |

## Règles DMZ (ordre obligatoire)

| # | Action | Source | Destination |
|---|---|---|---|
| 1 | Pass (ICMP) | DMZ subnets | This Firewall (self) |
| 2 | Block | DMZ subnets | INTERNAL_NETS |
| 3 | Pass | DMZ subnets | any |

## Ne pas publier
- `config.xml` (contient des hashes et clés)
- Mots de passe, clé CARP, certificats
