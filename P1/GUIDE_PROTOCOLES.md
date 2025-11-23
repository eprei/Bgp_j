# Guide des Protocoles de Routage - Pour Débutants

Ce guide explique les protocoles de routage utilisés dans ce projet et leur utilité.

## Table des matières

1. [Qu'est-ce que le routage ?](#quest-ce-que-le-routage)
2. [Zebra - Le gestionnaire de routage](#zebra---le-gestionnaire-de-routage)
3. [BGP (Border Gateway Protocol)](#bgp-border-gateway-protocol)
4. [OSPF (Open Shortest Path First)](#ospf-open-shortest-path-first)
5. [IS-IS (Intermediate System to Intermediate System)](#is-is-intermediate-system-to-intermediate-system)
6. [Quand utiliser quel protocole ?](#quand-utiliser-quel-protocole)
7. [Exemples pratiques](#exemples-pratiques)

---

## Qu'est-ce que le routage ?

Le **routage** est le processus qui permet de faire transiter des paquets de données d'un réseau à un autre. Imaginez que vous envoyez une lettre : le routage, c'est le système postal qui décide par quel chemin votre lettre va passer pour arriver à destination.

### Concepts de base

- **Routeur** : Un équipement qui connecte plusieurs réseaux et décide où envoyer les paquets
- **Table de routage** : Une liste qui indique au routeur vers où envoyer les paquets selon leur destination
- **Protocole de routage** : Un langage que les routeurs utilisent pour se parler et partager des informations sur les réseaux

### Types de routage

1. **Routage statique** : Les routes sont configurées manuellement (comme une carte routière fixe)
2. **Routage dynamique** : Les routes sont découvertes et mises à jour automatiquement par les protocoles

---

## Zebra - Le gestionnaire de routage

### À quoi sert Zebra ?

**Zebra** est le service de base de Quagga. Il ne fait pas de routage lui-même, mais il **gère** les autres protocoles de routage.

### Rôle de Zebra

- **Gestionnaire central** : Coordonne tous les protocoles de routage (BGP, OSPF, IS-IS)
- **Table de routage unifiée** : Combine toutes les routes apprises par les différents protocoles
- **Interface de configuration** : Permet de configurer les interfaces réseau et les routes statiques
- **Communication avec le noyau** : Transmet les routes au système d'exploitation

### Analogie

Zebra est comme le **chef d'orchestre** : il ne joue pas d'instrument, mais il coordonne tous les musiciens (les protocoles) pour créer une symphonie harmonieuse (le routage).

### Exemple

```bash
# Zebra gère les interfaces réseau
interface eth0
  ip address 192.168.1.1/24
```

---

## BGP (Border Gateway Protocol)

### À quoi sert BGP ?

**BGP** est le protocole utilisé pour le routage **entre différents opérateurs** (Autonomous Systems - AS). C'est le protocole qui fait fonctionner Internet.

### Caractéristiques principales

- **Routage inter-AS** : Connecte différents réseaux autonomes (opérateurs, entreprises)
- **Protocole de vecteur de chemin** : Choisit le meilleur chemin en fonction de politiques (pas seulement la distance)
- **Très stable** : Conçu pour gérer de très grands réseaux
- **Politique de routage** : Permet de choisir les routes selon des critères métier (coût, préférence, etc.)

### Quand utiliser BGP ?

✅ **Utilisez BGP quand :**
- Vous connectez votre réseau à Internet via plusieurs opérateurs
- Vous avez plusieurs sites distants avec des connexions multiples
- Vous voulez contrôler finement le trafic sortant/entrant
- Vous êtes un opérateur réseau ou un grand fournisseur de services

❌ **N'utilisez PAS BGP quand :**
- Vous avez un petit réseau local
- Vous n'avez qu'un seul lien vers Internet
- Vous voulez quelque chose de simple et rapide à configurer

### Analogie

BGP est comme le **système de navigation GPS international** : il connaît tous les pays (AS), les frontières (peering), et peut choisir le meilleur chemin en fonction de vos préférences (politiques).

### Exemple de configuration

```bash
# Configuration BGP pour un AS 65000
router bgp 65000
  bgp router-id 1.1.1.1
  network 192.168.1.0/24          # Annonce ce réseau
  neighbor 192.168.2.1 remote-as 65001  # Voisin dans un autre AS
```

### Cas d'usage réel

- **Fournisseur d'accès Internet (FAI)** : Utilise BGP pour échanger des routes avec d'autres FAI
- **Grande entreprise multi-sites** : Utilise BGP pour connecter plusieurs sites avec redondance
- **CDN (Content Delivery Network)** : Utilise BGP pour optimiser la livraison de contenu

---

## OSPF (Open Shortest Path First)

### À quoi sert OSPF ?

**OSPF** est un protocole de routage utilisé **à l'intérieur d'un même réseau** (intra-AS). Il trouve le chemin le plus court en fonction de la bande passante.

### Caractéristiques principales

- **Routage intra-AS** : Utilisé dans un seul réseau/entreprise
- **Protocole à état de liens** : Chaque routeur connaît la topologie complète du réseau
- **Calcul du chemin le plus court** : Utilise l'algorithme de Dijkstra
- **Convergence rapide** : S'adapte rapidement aux changements de topologie
- **Hiérarchique** : Utilise des "areas" (zones) pour réduire la complexité

### Quand utiliser OSPF ?

✅ **Utilisez OSPF quand :**
- Vous avez un réseau d'entreprise avec plusieurs routeurs
- Vous voulez une convergence rapide en cas de panne
- Vous avez besoin de redondance automatique
- Votre réseau est de taille moyenne à grande (plus de 3-4 routeurs)

❌ **N'utilisez PAS OSPF quand :**
- Vous avez un très petit réseau (2-3 routeurs)
- Vous préférez la simplicité (utilisez du routage statique)
- Vous avez besoin de routage entre différents opérateurs (utilisez BGP)

### Analogie

OSPF est comme un **GPS local** : il connaît toutes les rues de votre ville (réseau local), calcule le chemin le plus rapide, et se met à jour automatiquement si une route est fermée.

### Exemple de configuration

```bash
# Configuration OSPF pour un réseau local
router ospf
  ospf router-id 1.1.1.1
  network 192.168.1.0/24 area 0    # Réseau dans l'area 0 (backbone)
  network 10.0.0.0/24 area 1       # Réseau dans l'area 1
```

### Cas d'usage réel

- **Réseau d'entreprise** : Connecte plusieurs bâtiments ou sites
- **Campus universitaire** : Relie différents bâtiments
- **Datacenter** : Routage entre différents segments réseau

---

## IS-IS (Intermediate System to Intermediate System)

### À quoi sert IS-IS ?

**IS-IS** est un protocole de routage similaire à OSPF, mais basé sur le modèle OSI. Il est souvent utilisé dans les **grands réseaux d'opérateurs**.

### Caractéristiques principales

- **Protocole à état de liens** : Comme OSPF, connaît la topologie complète
- **Basé sur OSI** : Utilise le modèle OSI (pas TCP/IP)
- **Niveaux hiérarchiques** : Level-1 (local), Level-2 (backbone), Level-1-2 (les deux)
- **Très scalable** : Peut gérer de très grands réseaux
- **Utilisé par les opérateurs** : Très populaire chez les FAI et opérateurs télécoms

### Quand utiliser IS-IS ?

✅ **Utilisez IS-IS quand :**
- Vous êtes un opérateur réseau de grande taille
- Vous avez besoin d'une très grande scalabilité
- Vous voulez une alternative à OSPF
- Vous travaillez avec des réseaux MPLS

❌ **N'utilisez PAS IS-IS quand :**
- Vous avez un petit réseau d'entreprise (utilisez OSPF)
- Vous débutez (OSPF est plus simple)
- Vous n'avez pas besoin de scalabilité extrême

### Analogie

IS-IS est comme un **système de transport en commun sophistiqué** : très efficace pour les grandes villes (grands réseaux), avec des lignes locales (Level-1) et des lignes express (Level-2).

### Exemple de configuration

```bash
# Configuration IS-IS
router isis
  net 49.0001.0000.0000.0001.00    # Adresse réseau IS-IS
  is-type level-1-2                 # Fonctionne en Level-1 et Level-2

interface eth0
  ip router isis
  isis circuit-type level-1-2
```

### Cas d'usage réel

- **Réseaux d'opérateurs** : Backbone des grands FAI
- **Réseaux MPLS** : Utilisé avec MPLS pour le transport
- **Grandes entreprises** : Alternative à OSPF pour très grands réseaux

---

## Quand utiliser quel protocole ?

### Tableau de décision rapide

| Situation | Protocole recommandé | Pourquoi |
|-----------|----------------------|----------|
| Petit réseau local (2-3 routeurs) | Routage statique | Simple, pas besoin de protocole dynamique |
| Réseau d'entreprise (4-50 routeurs) | **OSPF** | Parfait pour réseaux moyens, facile à configurer |
| Très grand réseau d'entreprise | **OSPF** ou **IS-IS** | Les deux fonctionnent, OSPF plus simple |
| Réseau d'opérateur | **IS-IS** | Standard dans l'industrie |
| Connexion à Internet (plusieurs FAI) | **BGP** | Seul protocole pour inter-AS |
| Réseau multi-sites avec redondance | **BGP** | Contrôle fin des chemins |
| Datacenter | **OSPF** | Standard pour datacenters |

### Règle générale

1. **Réseau local/entreprise** → **OSPF**
2. **Connexion Internet/Inter-AS** → **BGP**
3. **Grand opérateur** → **IS-IS**
4. **Petit réseau** → **Routage statique**

---

## Exemples pratiques

### Exemple 1 : Petite entreprise

```
[Site A] ---- [Routeur 1] ---- [Routeur 2] ---- [Site B]
```

**Configuration recommandée : OSPF**

```bash
# Routeur 1
router ospf
  network 192.168.1.0/24 area 0
  network 10.0.0.0/24 area 0

# Routeur 2
router ospf
  network 10.0.0.0/24 area 0
  network 172.16.0.0/24 area 0
```

**Pourquoi OSPF ?** Simple, automatique, convergence rapide.

---

### Exemple 2 : Entreprise avec connexion Internet

```
[Internet] ---- [BGP] ---- [Routeur] ---- [OSPF] ---- [Réseau interne]
```

**Configuration :**
- **BGP** pour la connexion Internet (inter-AS)
- **OSPF** pour le réseau interne (intra-AS)

**Pourquoi les deux ?** BGP pour Internet, OSPF pour le réseau local.

---

### Exemple 3 : Topologie du projet

```
[wil-basic-1] ---- [wil-router-1] ---- [wil-router-2] ---- [wil-basic-2]
```

**Configuration possible :**

```bash
# wil-router-1
router ospf
  network 192.168.1.0/24 area 0
  network 10.0.0.0/24 area 0

router bgp 65000
  neighbor 10.0.0.2 remote-as 65001

# wil-router-2
router ospf
  network 10.0.0.0/24 area 0
  network 172.16.0.0/24 area 0

router bgp 65001
  neighbor 10.0.0.1 remote-as 65000
```

---

## Résumé des protocoles

| Protocole | Portée | Complexité | Scalabilité | Usage principal |
|-----------|--------|------------|-------------|-----------------|
| **Zebra** | Gestion | Faible | - | Gestionnaire de routage |
| **BGP** | Inter-AS | Élevée | Très élevée | Internet, multi-sites |
| **OSPF** | Intra-AS | Moyenne | Élevée | Réseaux d'entreprise |
| **IS-IS** | Intra-AS | Élevée | Très élevée | Opérateurs, MPLS |

---

## Commandes utiles pour comprendre

### Voir les routes apprises

```bash
vtysh
show ip route          # Toutes les routes (Zebra)
show ip bgp            # Routes BGP
show ip ospf database   # Base de données OSPF
show isis database      # Base de données IS-IS
```

### Voir les voisins

```bash
show ip bgp neighbors    # Voisins BGP
show ip ospf neighbor    # Voisins OSPF
show isis neighbor       # Voisins IS-IS
```

### Voir la configuration

```bash
show running-config     # Configuration actuelle
show startup-config     # Configuration sauvegardée
```

---

## Conclusion

- **Zebra** : Gère tout, ne fait pas de routage
- **BGP** : Pour Internet et connexions entre opérateurs
- **OSPF** : Pour réseaux d'entreprise (le plus courant)
- **IS-IS** : Pour grands opérateurs (alternative à OSPF)

Dans ce projet, vous avez les **trois protocoles actifs** pour apprendre et expérimenter. Commencez par **OSPF** pour comprendre les bases, puis explorez **BGP** et **IS-IS** selon vos besoins.

---

## Ressources pour aller plus loin

- **Documentation Quagga** : https://www.quagga.net/docs.php
- **RFC BGP** : RFC 4271
- **RFC OSPF** : RFC 2328
- **RFC IS-IS** : RFC 1195

