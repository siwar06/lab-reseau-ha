# 2. Vérifications Cisco

Pour chaque commande : lancer sur l'équipement, copier la sortie dans `outputs/<nom>.txt` (ou capture d'écran dans `images/`).

## Commandes demandées

| Équipement | Commande | Fichier | Attendu |
|---|---|---|---|
| Sw-core1 | `show standby brief` | `outputs/core1-standby.txt` | Active sur ses VLANs, Standby sur les autres |
| Sw-core2 | `show standby brief` | `outputs/core2-standby.txt` | Active sur ses VLANs, Standby sur les autres |
| Sw-core1 | `show etherchannel summary` | `outputs/core1-etherchannel.txt` | `Po11`, `Po12`, `Po20` en **(SU)** avec ports **(P)** |
| Sw-core2 | `show etherchannel summary` | `outputs/core2-etherchannel.txt` | `Po13`, `Po14`, `Po20` en **(SU)** |
| Sw-core1 | `show ip ospf neighbor` | `outputs/core1-ospf.txt` | Voisins en **FULL** |
| Sw-core2 | `show ip ospf neighbor` | `outputs/core2-ospf.txt` | Voisins en **FULL** |
| L3-edge1, L3-edge2 | `show ip ospf neighbor` | `outputs/edge1-ospf.txt`, `edge2-ospf.txt` | FULL avec cœurs et branche |

## Commandes complémentaires utiles

```
show ip interface brief | include Vlan
show vlan brief
show interfaces trunk
show lacp neighbor
show spanning-tree vlan 80
show ip route 0.0.0.0
show running-config interface vlan 80
show ip dhcp binding
show ip route ospf
```

## Comment enregistrer proprement
- PuTTY : *Session > Logging > Printable output* avant de lancer la session
- ou copier-coller la sortie dans un `.txt`

## Avant de publier
Supprimer toute ligne contenant `enable secret`, `enable password`, `username ... secret|password`, `snmp-server community`.
