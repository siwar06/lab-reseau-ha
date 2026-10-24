# 5. Problèmes rencontrés et solutions

## 5.1 `ping 8.8.8.8 source vlan 20` échouait
- **Symptôme** : le ping depuis un cœur avec la source VLAN 80 marchait, pas avec la source VLAN 20 ni depuis un PC.
- **Diagnostic** : routes statiques, alias, règles et NAT pfSense corrects. La capture sur pfSense montrait requête **et** réponse. Le ping pfSense → IP réelles des cœurs marchait, pfSense → VIP HSRP `10.1.80.1` échouait alors que l'ARP résolvait la MAC virtuelle `0000.0c07.ac50`.
- **Cause (probable)** : limitation de l'émulation IOL sur la MAC virtuelle HSRP du VLAN 80.
- **Solution** : `standby 80 use-bia` sur `interface vlan 80` des deux cœurs (HSRP utilise la MAC réelle).

## 5.2 Isolation DMZ inefficace
- **Cause** : règle *Pass DMZ → any* placée **avant** la règle *Block DMZ → INTERNAL_NETS* (pfSense applique la première règle qui correspond).
- **Solution** : ordre ICMP vers *This Firewall*, puis Block, puis Pass any.

## 5.3 Ping de la VIP DMZ bloqué
- **Cause** : `DMZ address` ne couvre pas la VIP CARP, et `INTERNAL_NETS` (10.1.0.0/16) contient la DMZ.
- **Solution** : destination **This Firewall (self)**, protocole ICMP seulement (évite l'accès au GUI/SSH de pfSense depuis la DMZ).

## 5.4 Accès au GUI de pfSense2
- **Cause** : tentative en `http://` ; le GUI est en HTTPS.
- **Solution** : `https://<IP WAN>`.

## 5.5 Config perdue après redémarrage
- **Leçon** : `write memory` sur tous les équipements IOL et `save` sur les VPCS avant d'arrêter EVE-NG.
