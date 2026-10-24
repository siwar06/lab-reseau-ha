# 4. Tests ping et DMZ

Remplir la colonne **Résultat** (✅ / ❌) et joindre une capture dans `images/04-<nom>.png`.

## Connectivité LAN et Internet

| # | Depuis | Commande | Attendu | Résultat |
|---|---|---|---|---|
| 1 | PC VLAN 10 | `ping 10.1.10.1` | ✅ | ☐ |
| 2 | PC VLAN 20 (PC1-salle1) | `ping 10.1.20.1` | ✅ | ☐ |
| 3 | PC VLAN 30 | `ping 10.1.30.1` | ✅ | ☐ |
| 4 | PC VLAN 10 | `ping 8.8.8.8` | ✅ | ☐ |
| 5 | PC VLAN 20 | `ping 8.8.8.8` | ✅ | ☐ |
| 6 | PC VLAN 30 | `ping 8.8.8.8` | ✅ | ☐ |
| 7 | PC VLAN 20 | `ping 10.1.80.4` (VIP CARP) | ✅ | ☐ |
| 8 | Sw-core1 | `ping 8.8.8.8 source vlan 20` | ✅ | ☐ |
| 9 | PC branche (PC27 / PC28) | `ping 8.8.8.8` | ✅ | ☐ |

## DMZ

| # | Depuis | Commande | Attendu | Résultat |
|---|---|---|---|---|
| 10 | Serveur_DNS | `ping 10.1.100.1` (VIP) | ✅ | ☐ |
| 11 | Serveur_DNS | `ping 8.8.8.8` | ✅ (NAT) | ☐ |
| 12 | Serveur_DNS | `ping 10.1.20.101` (cœur) | ❌ (**isolation voulue**) | ☐ |
| 13 | PC VLAN 20 | `ping 10.1.100.3` | ✅ (LAN → DMZ autorisé) | ☐ |

## Haute disponibilité

| # | Test | Attendu | Résultat |
|---|---|---|---|
| 14 | `ping 8.8.8.8 -t` depuis un PC, puis *Maintenance Mode* CARP sur pfSense1 | pfSense2 MASTER, 3 à 5 pertes puis reprise | ☐ |
| 15 | `shutdown` de `Vlan 20` sur Sw-core2 | Sw-core1 devient Active VL20, le PC garde l'accès | ☐ |
| 16 | Arrêt d'un lien d'un EtherChannel | Le lien reste up (port restant), OSPF reste FULL | ☐ |

## Après chaque test de panne
Remettre l'état d'origine (`no shutdown`, *Leave Maintenance Mode*).
