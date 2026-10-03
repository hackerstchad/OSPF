# `OSPF` — Open Shortest Path First

```
  ___  ____   ____ ____  _____ 
 / _ \/ ___| / ___|  _ \|  ___|
| | | \___ \| |   | |_) | |_   
| |_| |___) | |___|  __/|  _|  
 \___/|____/ \____|_|   |_|    
```

> **Le protocole de routage à état de lien le plus déployé au monde.**  
> Conçu, écrit et présenté par **`hackers_tchad`** — pour les hackers, les ingénieurs réseau et les curieux du monde entier.

---

## 🧬 Vue d'ensemble

| Propriété | Valeur |
|-----------|--------|
| **Nom complet** | Open Shortest Path First |
| **Type** | Protocole de routage IGP (Interior Gateway Protocol) |
| **Algorithme** | Dijkstra / SPF (Shortest Path First) |
| **Famille** | Link-State (à état de lien) |
| **Protocole IP** | 89 |
| **Multicast** | `224.0.0.5` (AllSPFRouters), `224.0.0.6` (AllDRouters) |
| **Standard** | RFC 2328 (OSPFv2), RFC 5340 (OSPFv3) |
| **Auteur principal** | John T. Moy (1991) |
| **Créé par** | IETF (Internet Engineering Task Force) |
| **Présenté par** | `hackers_tchad` |

---

## 📚 Table des matières

1. [Introduction](#introduction)
2. [Historique](#historique)
3. [Versions d'OSPF](#versions-dospf)
4. [Principes fondamentaux](#principes-fondamentaux)
5. [Types de paquets OSPF](#types-de-paquets-ospf)
6. [Les LSA (Link-State Advertisements)](#les-lsa-link-state-advertisements)
7. [Les zones OSPF](#les-zones-ospf)
8. [Les types de routeurs](#les-types-de-routeurs)
9. [Les états des voisins](#les-états-des-voisins)
10. [DR / BDR](#dr--bdr)
11. [Métrique OSPF (Cost)](#métrique-ospf-cost)
12. [Authentification](#authentification)
13. [Configuration Cisco](#configuration-cisco)
14. [Configuration Juniper](#configuration-juniper)
15. [Configuration Huawei](#configuration-huawei)
16. [OSPFv3](#ospfv3)
17. [OSPF multi-area](#ospf-multi-area)
18. [OSPF over NBMA](#ospf-over-nbma)
19. [Dépannage et commandes](#dépannage-et-commandes)
20. [Comparaison avec d'autres protocoles](#comparaison-avec-dautres-protocoles)
21. [Cmatrix et style terminal](#cmatrix-et-style-terminal)
22. [Ressources, livres et liens](#ressources-livres-et-liens)
23. [Glossaire](#glossaire)
24. [Conclusion](#conclusion)

---

## 🌍 Introduction

**OSPF** est un protocole de routage **intérieur (IGP)** qui utilise l'algorithme de **Dijkstra** pour calculer le plus court chemin dans un graphe de réseau. Contrairement aux protocoles à vecteur de distance (RIP, EIGRP ancien), OSPF connaît la **topologie complète** du réseau grâce aux **LSA**.

### Pourquoi OSPF ?

- Convergence rapide
- Pas de limite de sauts (contrairement à RIP)
- Support du VLSM et du CIDR
- Hiérarchisation par zones
- Sécurisable par authentification
- Standard ouvert (RFC)

### Citation

> "OSPF est le cerveau cartographique de l'Internet interne."
> — `hackers_tchad`

---

## 🕰️ Historique

| Année | Événement |
|-------|-----------|
| 1987 | Première discussion IETF sur un successeur de RIP |
| 1989 | Publication OSPFv1 dans le RFC 1131 |
| 1991 | OSPFv2 standardisé dans le RFC 1247, puis RFC 2328 |
| 1994 | Ajout de l'authentification MD5 (RFC 2154) |
| 1997 | Support explicite de NSSA (RFC 3101) |
| 1998 | RFC 2328 devient le standard définitif d'OSPFv2 |
| 2008 | OSPFv3 pour IPv6 standardisé (RFC 5340) |
| 2010+ | Extensions pour MPLS, Traffic Engineering, Segment Routing |

### Créateurs

- **John T. Moy** : concepteur principal d'OSPF
- **IETF OSPF Working Group**
- Documentation et vulgarisation par **`hackers_tchad`**

---

## 🔢 Versions d'OSPF

### OSPFv2

- IPv4 uniquement
- RFC 2328
- Le plus déployé

### OSPFv3

- IPv6 natif
- RFC 5340
- Utilise des LSA modifiés
- Peut être adapté pour IPv4 (RFC 5838)

### Comparaison rapide

| Caractéristique | OSPFv2 | OSPFv3 |
|-----------------|--------|--------|
| Adressage | IPv4 | IPv6 |
| Protocole IP | 89 | 89 |
| Multicast | 224.0.0.5 / .6 | FF02::5 / FF02::6 |
| Authentification | MD5, texte | IPsec natif |
| LSA | Types 1 à 7 | Types 0x2001 à 0x2009 |

---

## ⚙️ Principes fondamentaux

### États de lien

Chaque routeur OSPF maintient une **LSDB (Link-State Database)** identique à celle des autres routeurs de la même zone.

### Processus

1. Découverte des voisins (Hello)
2. Échange des LSDB (DBD / LSR / LSU / LSAck)
3. Exécution de l'algorithme de Dijkstra
4. Calcul de la table de routage

### Arbre SPF

L'algorithme construit un **arbre sans boucle** dont la racine est le routeur local.

---

## 📦 Types de paquets OSPF

| Type | Nom | Description |
|------|-----|-------------|
| 1 | Hello | Découverte et maintien des voisins |
| 2 | DBD (Database Description) | Résumé de la LSDB |
| 3 | LSR (Link-State Request) | Demande d'un LSA spécifique |
| 4 | LSU (Link-State Update) | Envoi d'un ou plusieurs LSA |
| 5 | LSAck (Link-State Ack) | Accusé de réception |

### Format du paquet Hello

```
+-----------------------------------+
| Version | Type | Packet Length    |
+-----------------------------------+
|           Router ID               |
+-----------------------------------+
|           Area ID                 |
+-----------------------------------+
| Checksum | Auth Type | Auth Data   |
+-----------------------------------+
| Network Mask | HelloInt | Options  |
+-----------------------------------+
| Router Priority | Dead Interval    |
+-----------------------------------+
|      Designated Router            |
+-----------------------------------+
|    Backup Designated Router       |
+-----------------------------------+
|       Neighbor Router IDs         |
+-----------------------------------+
```

---

## 🧩 Les LSA (Link-State Advertisements)

| Type | Nom | Description |
|------|-----|-------------|
| 1 | Router LSA | Généré par chaque routeur dans une zone |
| 2 | Network LSA | Généré par le DR sur un segment broadcast |
| 3 | Summary LSA (IP Network) | Généré par un ABR pour annoncer des routes inter-zones |
| 4 | Summary LSA (ASBR) | Généré par un ABR pour localiser l'ASBR |
| 5 | External LSA | Généré par un ASBR pour des routes externes |
| 7 | NSSA External LSA | Généré par un ASBR dans une zone NSSA |

### LSA OSPFv3

| Code | Nom |
|------|-----|
| 0x2001 | Router-LSA |
| 0x2002 | Network-LSA |
| 0x2003 | Inter-Area-Prefix-LSA |
| 0x2004 | Inter-Area-Router-LSA |
| 0x4005 | AS-External-LSA |
| 0x2009 | Intra-Area-Prefix-LSA |

---

## 🗺️ Les zones OSPF

### Types de zones

| Zone | Description |
|------|-------------|
| Backbone (Area 0) | Zone centrale obligatoire |
| Standard Area | Zone normale recevant toutes les routes |
| Stub Area | Ne reçoit pas de routes externes (type 5) |
| Totally Stubby | Ne reçoit que la route par défaut |
| NSSA (Not-So-Stubby Area) | Autorise les ASBR locaux avec LSA type 7 |
| Totally NSSA | NSSA + totally stubby |

### Règles de conception

- Toutes les zones doivent être connectées à Area 0
- Un ABR relie Area 0 à une autre zone
- Un virtual-link peut contourner une déconnexion temporaire

---

## 🖥️ Les types de routeurs

| Type | Abréviation | Rôle |
|------|-------------|------|
| Internal Router | IR | Toutes ses interfaces dans la même zone |
| Area Border Router | ABR | Interfaces dans Area 0 et une autre zone |
| Autonomous System Boundary Router | ASBR | Injecte des routes externes dans OSPF |
| Backbone Router | BR | Au moins une interface dans Area 0 |

---

## 🔁 Les états des voisins

| État | Description |
|------|-------------|
| Down | Aucun Hello reçu |
| Init | Hello reçu, mais pas de Router ID local dedans |
| 2-Way | Communication bidirectionnelle établie |
| ExStart | Négociation du master/slave pour le DBD |
| Exchange | Échange des DBD |
| Loading | Requête/réponse des LSA manquants |
| Full | LSDB synchronisée |

### Conditions pour Full

- Mêmes intervalles Hello/Dead
- Même masque de sous-réseau (sur broadcast)
- Même zone OSPF
- Mêmes options (stub flag)
- Authentification OK

---

## 🏆 DR / BDR

### Pourquoi un DR ?

Sur un réseau multi-accès (Ethernet), OSPF élit un **Designated Router** pour réduire le nombre d'adjacences.

### Calcul

```
DR = routeur avec la plus haute priorité OSPF
BDR = deuxième plus haute priorité
```

### Commandes Cisco

```cisco
interface GigabitEthernet0/0
 ip ospf priority 10
```

### Priorité par défaut

- Cisco : 1
- 0 = jamais DR/BDR

---

## 📏 Métrique OSPF (Cost)

```
Cost = Reference Bandwidth / Interface Bandwidth
```

### Valeurs par défaut

| Interface | Bande passante | Cost par défaut |
|-----------|----------------|-----------------|
| Ethernet | 10 Mbps | 10 |
| FastEthernet | 100 Mbps | 1 |
| GigabitEthernet | 1 Gbps | 1 |
| 10 GigabitEthernet | 10 Gbps | 1 |

### Modifier la bandwidth de référence

```cisco
router ospf 1
 auto-cost reference-bandwidth 10000
```

### Forcer un cost manuel

```cisco
interface GigabitEthernet0/0
 ip ospf cost 50
```

---

## 🔐 Authentification

### Types

| Type | Description |
|------|-------------|
| Null | Aucune authentification |
| Texte | Mot de passe en clair (RFC 2328) |
| MD5 | Hachage MD5 par paquet (RFC 2328) |
| HMAC-SHA | SHA-1, SHA-256, SHA-384, SHA-512 (RFC 7474) |

### Configuration MD5

```cisco
interface GigabitEthernet0/0
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 MON_MDP
```

---

## ⚙️ Configuration Cisco

### Configuration de base

```cisco
! Activer OSPF process ID 1
router ospf 1
 router-id 1.1.1.1
 network 192.168.1.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0

! Interface
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 ip ospf 1 area 0
 ip ospf cost 10
 ip ospf priority 5
```

### Multi-area

```cisco
router ospf 1
 router-id 2.2.2.2
 network 192.168.1.0 0.0.0.255 area 0
 network 192.168.2.0 0.0.0.255 area 1
 network 192.168.3.0 0.0.0.255 area 2
```

### ASBR avec route redistribuée

```cisco
router ospf 1
 redistribute static subnets
 default-information originate
```

### Stub Area

```cisco
router ospf 1
 area 1 stub
```

### NSSA

```cisco
router ospf 1
 area 1 nssa
```

---

## ⚙️ Configuration Juniper

```junos
protocols {
    ospf {
        area 0.0.0.0 {
            interface ge-0/0/0.0 {
                priority 10;
                metric 50;
            }
            interface lo0.0 {
                passive;
            }
        }
        area 0.0.0.1 {
            interface ge-0/0/1.0;
        }
    }
}
```

---

## ⚙️ Configuration Huawei

```huawei
ospf 1 router-id 1.1.1.1
 area 0
  network 192.168.1.0 0.0.0.255
  network 10.0.0.0 0.0.0.3
 interface GigabitEthernet0/0/1
  ospf cost 10
  ospf dr-priority 5
```

---

## 🌐 OSPFv3

### Configuration Cisco OSPFv3

```cisco
ipv6 unicast-routing
!
interface GigabitEthernet0/0
 ipv6 address 2001:DB8::1/64
 ipv6 ospf 1 area 0
!
ipv6 router ospf 1
 router-id 1.1.1.1
```

### Différences clés

- Séparation topologie/adressage
- LSA modifiés
- Authentification via IPsec

---

## 🌐 OSPF Multi-Area

### Pourquoi segmenter ?

- Réduire la taille de la LSDB
- Limiter les calculs SPF
- Contenir les instabilités
- Faciliter la hiérarchisation

### Règles

- Toutes les zones doivent toucher Area 0
- Un ABR génère des Summary LSA (type 3)
- Les routes inter-zones sont préfixées par IA (Inter-Area)

---

## 🌐 OSPF over NBMA

### Modes NBMA

| Mode | Description |
|------|-------------|
| NBMA | Nécessite configuration manuelle des voisins |
| Point-to-Point | Un sous-interface par voisin |
| Point-to-Multipoint | Traite chaque PVC comme un lien point-à-point |
| Broadcast | Si le cloud supporte la diffusion |

### Configuration voisin manuel

```cisco
router ospf 1
 neighbor 10.0.0.2
 neighbor 10.0.0.3
```

---

## 🛠️ Dépannage et commandes

### Cisco

```cisco
show ip ospf neighbor
show ip ospf neighbor detail
show ip ospf database
show ip ospf interface brief
show ip route ospf
show ip ospf statistics
show ip ospf border-routers
show ip ospf virtual-links
show ip ospf request-list
show ip ospf retransmission-list
show ip ospf events
```

### Juniper

```junos
show ospf neighbor
show ospf interface
show ospf database
show route protocol ospf
```

### Huawei

```huawei
display ospf peer
display ospf interface
display ospf lsdb
display ospf routing
display ospf error
```

### Diagnostic Windows

Sous Windows, OSPF n'est pas natif. Utilisez :

- **Wireshark** pour capturer les paquets IP 89
- **Packet Tracer / GNS3 / EVE-NG** pour simuler
- **PowerShell** pour vérifier la table de routage :

```powershell
Get-NetRoute -AddressFamily IPv4
```

---

## ⚔️ Comparaison avec d'autres protocoles

| Critère | OSPF | RIP | EIGRP | IS-IS |
|---------|------|-----|-------|-------|
| Type | Link-State | Distance-Vector | Hybride | Link-State |
| Convergence | Rapide | Lente | Très rapide | Rapide |
| Standard | Oui (RFC) | Oui | Propriétaire Cisco | Oui (ISO) |
| Métrique | Cost (bande passante) | Sauts | Composite | Cost |
| Support IPv6 | OSPFv3 | RIPng | Oui | Oui |
| Complexité | Moyenne | Faible | Moyenne | Moyenne |
| Scalabilité | Bonne | Faible | Bonne | Excellente |

---

## 💻 Cmatrix et style terminal

Pour visualiser OSPF en mode "matrix" vert dans un terminal Linux/macOS/Windows WSL :

```bash
sudo apt install cmatrix
cmatrix -C green -u 2
```

### Script Python cmatrix-style

```python
import random, shutil, time, sys

cols, _ = shutil.get_terminal_size()
chars = "01ABCDEFOSPFv2v3DRBDRABRASBR"
drops = [0] * cols

while True:
    line = ""
    for i in range(cols):
        if random.random() > 0.95:
            drops[i] = random.randint(0, 20)
        if drops[i] > 0:
            line += f"\033[32m{random.choice(chars)}\033[0m"
            drops[i] -= 1
        else:
            line += " "
    sys.stdout.write(line + "\r")
    sys.stdout.flush()
    time.sleep(0.05)
```

---

## 📖 Ressources, livres et liens

### Livres

| Auteur | Titre | Année |
|--------|-------|-------|
| John T. Moy | OSPF: Anatomy of an Internet Routing Protocol | 1998 |
| Jeff Doyle | Routing TCP/IP, Volume I | 2001 |
| Thomas M. Thomas II | OSPF Network Design Solutions | 2003 |
| Sam Halabi | Internet Routing Architectures | 2000 |

### RFCs essentiels

- RFC 2328 — OSPF Version 2
- RFC 5340 — OSPF for IPv6 (OSPFv3)
- RFC 3101 — The OSPF Not-So-Stubby Area (NSSA) Option
- RFC 2154 — OSPF with Digital Signatures
- RFC 7474 — Security Extension for OSPFv2

### Liens utiles

- https://datatracker.ietf.org/wg/ospf/documents/
- https://www.cisco.com/c/en/us/support/ip/open-shortest-path-first-ospf/ser-routing-products.html
- https://www.juniper.net/documentation/en_US/junos/topics/concept/ospf-overview.html
- https://en.wikipedia.org/wiki/Open_Shortest_Path_First

---

## 📖 Glossaire

| Terme | Définition |
|-------|------------|
| LSA | Link-State Advertisement |
| LSDB | Link-State Database |
| SPF | Shortest Path First |
| ABR | Area Border Router |
| ASBR | AS Boundary Router |
| DR | Designated Router |
| BDR | Backup Designated Router |
| NSSA | Not-So-Stubby Area |
| Stub | Zone qui ne reçoit pas de routes externes |
| Cost | Métrique basée sur la bande passante |

---

## ✅ Conclusion

OSPF reste le protocole IGP de référence pour les réseaux d'entreprise et les FAI. Sa robustesse, son ouverture et sa scalabilité en font un incontournable pour tout ingénieur réseau.

> **Protocole maîtrisé = réseau maîtrisé.**  
> — `hackers_tchad`

```
   ____  ____  _________  ________________  ____  ___   ____________  ______
  / __ \/ __ \/ ____/   |/_  __/ ____/ __ )/ __ \/   | / ____/ __ \ \/ / __ \
 / / / / /_/ / /_  / /| | / / / __/ / __  / / / / /| |/ / __/ /_/ /\  / / / /
/ /_/ / ____/ __/ / ___ |/ / / /___/ /_/ / /_/ / ___ / /_/ / _, _/ / / /_/ /
\____/_/   /_/   /_/  |_/_/ /_____/_____/_____/_/  |_\____/_/ |_| /_/_____/
```

---

*Document créé par `hackers_tchad` — pour l'éducation, la recherche et l'administration réseau.*
