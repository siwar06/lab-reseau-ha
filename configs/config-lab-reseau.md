# Configuration du lab réseau : récapitulatif complet

Dernière mise à jour : 18/10/2026 (EVE-NG sous VMware, pfSense Plus 26.07)

> Les blocs de commandes sont à coller tels quels dans la console de chaque équipement.
> Les mots de passe ne sont volontairement pas écrits dans ce fichier.
> Les points marqués **(à vérifier)** n'ont pas été confirmés par une capture pendant nos échanges.

---

## 1. Vue d'ensemble

### 1.1 Équipements

| Équipement | Rôle |
|---|---|
| Sw-core1, Sw-core2 | Cœur L3 redondant : HSRP (load-sharing), DHCP en split-scope, OSPF |
| L2-SW-Salle1, L2-SW-SalleTP, L2-SW-salleEnseignants, L2-Sw-Bibliotheque | Switchs d'accès (VLANs 10, 20, 30) |
| L3-edge1, L3-edge2 | Routeurs de bordure : OSPF, liens série vers les branches |
| Branche1, Branche2 | Sites distants (liens série 80.0.0.0/8 et 90.0.0.0/8) |
| pfSense1, pfSense2 | Pare-feu en haute disponibilité (CARP + pfsync), NAT vers Internet |
| PC27, PC28 | Postes des branches (adresses statiques) |

### 1.2 VLANs

| VLAN | Nom | Réseau | VIP HSRP (passerelle) |
|---|---|---|---|
| 10 | Etudiants | 10.1.10.0/24 | 10.1.10.1 |
| 20 | Enseignants | 10.1.20.0/24 | 10.1.20.1 |
| 30 | Administration | 10.1.30.0/24 | 10.1.30.1 |
| 80 | Transit (cœur ↔ pfSense) | 10.1.80.0/24 | 10.1.80.1 |
| 40 / 50 / 60 / 70 | Liens core ↔ edge | voir 1.4 | aucune |

### 1.3 Répartition HSRP (load-sharing, preempt actif)

| VLAN | Active (priorité 110) | Standby (priorité 100) |
|---|---|---|
| 10 | Sw-core1 | Sw-core2 |
| 20 | Sw-core2 | Sw-core1 |
| 30 | Sw-core1 | Sw-core2 |
| 80 | Sw-core2 | Sw-core1 |

Si un core tombe, l'autre devient Active pour tous les VLANs. Au retour du core, `preempt` lui rend ses VLANs.

### 1.4 Liens core ↔ edge (EtherChannel LACP en L2 + interface VLAN)

| Lien | Ports core | Ports edge | VLAN | Groupe | IP core | IP edge |
|---|---|---|---|---|---|---|
| core1 ↔ edge1 | core1 e1/0-1 | edge1 e0/0-1 | 40 | 11 | 40.0.0.1/8 | 40.0.0.2/8 |
| core1 ↔ edge2 | core1 e1/2-3 | edge2 e0/0-1 | 70 | 12 | 70.0.0.1/8 | 70.0.0.2/8 |
| core2 ↔ edge2 | core2 e1/0-1 | edge2 e0/2-3 | 60 | 13 | 60.0.0.1/8 | 60.0.0.2/8 |
| core2 ↔ edge1 | core2 e1/2-3 | edge1 e0/2-3 | 50 | 14 | 50.0.0.1/8 | 50.0.0.2/8 |

Autres liens : core1 e2/1-3 ↔ core2 e2/1-3 (Port-channel 20, trunk VLAN 80) ; core1 e2/0 ↔ pfSense1 e2 et core2 e2/0 ↔ pfSense2 e2 (ports d'accès VLAN 80) ; edge1 s2/0 ↔ Branche1 s1/0 (80.0.0.0/8) ; edge2 s2/0 ↔ Branche2 s1/0 (90.0.0.0/8).

### 1.5 OSPF (area 0)

| Routeur | Router-ID | Réseaux annoncés |
|---|---|---|
| Sw-core1 | 1.1.1.1 | 10.1.10/20/30.0, 40.0.0.0/8, 70.0.0.0/8 |
| Sw-core2 | 2.2.2.2 | 10.1.10/20/30.0, 50.0.0.0/8, 60.0.0.0/8 |
| L3-edge1 | 3.3.3.3 | 40, 50, 80.0.0.0/8 |
| L3-edge2 | 4.4.4.4 | 60, 70, 90.0.0.0/8 |
| Branche1 | 5.5.5.5 | 80.0.0.0/8, 192.168.2.0/24 |
| Branche2 | 6.6.6.6 | 90.0.0.0/8, 192.168.1.0/24 |

Les cores diffusent une route par défaut (`default-information originate`) vers `10.1.80.4`, la VIP CARP de pfSense.

---

## 2. Switchs d'accès (inchangés depuis la v5)

### 2.1 L2-SW-Salle1 (uplinks : e0/2 → core1 e0/0, e0/3 → core2 e0/0)
```
enable
configure terminal
hostname L2-SW-Salle1
vlan 10
 name Etudiants
vlan 20
 name Enseignants
interface range e0/2 - 3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20
interface e0/0
 switchport mode access
 switchport access vlan 20
interface e0/1
 switchport mode access
 switchport access vlan 10
end
write memory
```

### 2.2 L2-SW-SalleTP (uplinks : e0/3 → core1 e0/1, e1/0 → core2 e0/1)
```
enable
configure terminal
hostname L2-SW-SalleTP
vlan 10
 name Etudiants
vlan 20
 name Enseignants
vlan 30
 name Administration
interface e0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
interface e1/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
interface e0/0
 switchport mode access
 switchport access vlan 20
interface e0/1
 switchport mode access
 switchport access vlan 10
interface e0/2
 switchport mode access
 switchport access vlan 30
end
write memory
```

### 2.3 L2-SW-salleEnseignants (uplinks : e0/2 → core1 e0/2, e0/3 → core2 e0/2)
```
enable
configure terminal
hostname L2-SW-Ens
vlan 30
 name Administration
interface range e0/2 - 3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 30
interface e0/0
 switchport mode access
 switchport access vlan 30
interface e0/1
 switchport mode access
 switchport access vlan 30
end
write memory
```

### 2.4 L2-Sw-Bibliotheque (uplinks : e0/2 → core1 e0/3, e0/3 → core2 e0/3)
```
enable
configure terminal
hostname L2-Sw-Biblio
vlan 10
 name Etudiants
interface range e0/2 - 3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10
interface e0/0
 switchport mode access
 switchport access vlan 10
interface e0/1
 switchport mode access
 switchport access vlan 10
end
write memory
```

---

---

## 3. Mises à jour (21-23/10/2026)

### 3.1 Cœurs : VLAN 80 (correctif MAC HSRP)
```
interface vlan 80
 standby 80 ip 10.1.80.1
 standby use-bia
 standby 80 preempt delay minimum 60
```
Priorité VLAN 80 : core1 = 120 (Active), core2 = 110 (Standby) à la date de rédaction.
Route par défaut des cœurs : `ip route 0.0.0.0 0.0.0.0 10.1.80.4`.

### 3.2 R20 (switch DMZ)
```
hostname R20
vlan 100
 name DMZ
interface range e0/0 - 3
 switchport mode access
 switchport access vlan 100
 spanning-tree portfast
```

### 3.3 Hôtes DMZ
```
Serveur_DNS (VPCS) : ip 10.1.100.3/24 10.1.100.1
Hôte « Linux »     : 10.1.100.2/24, passerelle 10.1.100.1 (image IOL utilisée comme hôte de test)
```

### 3.4 pfSense (résumé)
| Élément | Valeur |
|---|---|
| LAN (transit) | pfSense1 10.1.80.2, pfSense2 10.1.80.3, VIP CARP 10.1.80.4 (VHID 80) |
| SYNC | 200.200.200.1 / 200.200.200.2 (pfsync + XMLRPC) |
| DMZ | pfSense1 10.1.100.253, pfSense2 10.1.100.252, VIP CARP 10.1.100.1 (VHID 100) |
| Gateway | `CORE_HSRP` = 10.1.80.1 |
| Routes statiques | 10.1.10/20/30.0/24, 192.168.1.0/24, 192.168.2.0/24 via `CORE_HSRP` |
| Alias | `INTERNAL_NETS` = 10.1.0.0/16, 192.168.1.0/24, 192.168.2.0/24 |
| NAT sortant | Hybrid : WAN address pour 10.1.0.0/16, 192.168.1.0/24, 192.168.2.0/24, 10.1.100.0/24 |
| Règles DMZ | 1) Pass ICMP → This Firewall · 2) Block → INTERNAL_NETS · 3) Pass → any |
