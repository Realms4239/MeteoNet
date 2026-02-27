# 🌩️ MeteoNet HCI : Infrastructure Réseau Hyperconvergée & NAC

![Statut](https://img.shields.io/badge/Statut-En_Production-success)
![Hyperviseur](https://img.shields.io/badge/Hyperviseur-Proxmox_VE_8-orange)
![Routeur](https://img.shields.io/badge/Routeur-pfSense_2.7-red)
![Licence](https://img.shields.io/badge/Licence-Open_Source-blue)

Ce dépôt contient l'architecture, la configuration et les codes de déploiement du projet **MeteoNet HCI**. Il s'agit d'une refonte complète d'une infrastructure réseau contrainte (matériel legacy, forte latence, CGNAT) vers un système Hyperconvergé (HCI) sécurisé, segmenté et hautement observable.

## 📑 Sommaire
- [Contexte & Problématique](#-contexte--problématique)
-[Architecture & Piliers](#-architecture--piliers)
- [Topologie et Plan d'Adressage](#-topologie-et-plan-dadressage)
- [Structure de ce Dépôt](#-structure-de-ce-dépôt)
- [Guide de Déploiement](#-guide-de-déploiement-usage)
-[Auteur](#-auteur)

---

## 🌍 Contexte & Problématique
Déployé dans le contexte insulaire de Madagascar, ce projet répond à une triple contrainte :
1. **Ressources limitées** : Utilisation d'un serveur unique Dell PowerEdge T410 (6 Go RAM).
2. **Instabilité WAN (Bufferbloat)** : Lignes à latence variable saturées par les téléchargements (réseau plat).
3. **Opacité FAI (CGNAT)** : Impossibilité d'utiliser des redirections de ports standards pour l'administration distante.

**La Solution** : L'ingénierie logicielle au service du matériel. Utilisation de la virtualisation de Type-1 pour encapsuler le routage, la sécurité, l'identité et la télémétrie en un seul nœud.

---

## 🏛️ Architecture & Piliers

Le système est construit sur **5 Piliers d'Ingénierie** :

1. **Sécurité (Layer 2/3)** : Segmentation VLAN (802.1Q), Port Security dynamique (Cisco), et politiques de pare-feu stricts "Zero Trust" (pfSense).
2. **Stabilité (QoS)** : Éradication du Bufferbloat via des limiters `dummynet` utilisant l'algorithme **FQ_CoDel**.
3. **Identité (NAC)** : Portail Captif adossé à **FreeRADIUS** avec promotion dynamique des flux Staff vs Guest (Attributs WISPr).
4. **Efficience (Ops-Core)** : Déploiement de micro-services en Go (AdGuard Home) et C (Netdata) via des conteneurs LXC/Docker pour minimiser l'empreinte mémoire.
5. **Mobilité (VPN)** : Implémentation d'un réseau Overlay (Tailscale) pour le NAT Traversal et l'accès distant sécurisé.

---

## 🗺️ Topologie et Plan d'Adressage

![Schéma de Topologie](./docs/images/topologie.png) *(Note: Ajouter votre image dans le dossier docs)*

| Interface / Zone | VLAN | Sous-réseau (CIDR) | Sécurité Physique |
| :--- | :--- | :--- | :--- |
| **Management** | `1` | `10.0.0.0/24` | Accès via Port 16 (Tech Port) |
| **Staff (Sécurisé)**| `10` | `10.0.10.0/24` | Port Security (Max 2 MAC) |
| **Guest (Wi-Fi)** | `20` | `10.0.20.0/24` | Private VLAN Edge (Isolation) |
| **VoIP (Futur)** | `40` | `10.0.40.0/24` | Priorité FQ_CoDel Absolue |

---

## 📂 Structure de ce Dépôt

* `/cisco/` : Script de configuration CLI "One-Shot" pour le commutateur Catalyst 1000.
* `/pfsense/` : 
  * Fichier `.xml` de restauration de la configuration globale.
  * Code HTML/CSS/JS du Portail Captif personnalisé (double palier d'authentification).
* `/ops-core/` : 
  * `docker-compose.yml` de la stack d'observabilité (AdGuard, Netdata, Uptime Kuma).
  * Fichiers de configuration du Reverse Proxy Nginx (Routage par Host Header).

---

## 🚀 Guide de Déploiement (Usage)

Pour reproduire cet environnement, suivez cet ordre strict :

### 1. Préparation de l'Hôte (Proxmox VE)
1. Installez Proxmox VE sur le serveur hôte.
2. Configurez le stockage en **LVM-Thin** (pour économiser la RAM face à ZFS).
3. Créez un bridge virtuel `vmbr1` asservi à l'interface physique LAN et activez impérativement l'option **VLAN-Aware**.

### 2. Configuration du Switch (Layer 2)
1. Connectez-vous en console au switch Cisco.
2. Copiez-collez le contenu de `cisco/cisco_catalyst_gold_config.txt`.
3. Reliez le port 1 (Trunk) au serveur Proxmox.

### 3. Déploiement du Cœur de Réseau (pfSense)
1. Créez une VM Proxmox (2 vCPU, 2.5 Go RAM) et installez pfSense.
2. Restaurez la configuration via le fichier `pfsense/pfsense_backup_sanitized.xml`.
3. **Attention** : Désactivez le *Hardware Checksum Offloading* dans les paramètres avancés pour garantir la stabilité des interfaces VirtIO.
4. Uploadez `portal.html` dans les paramètres du Captive Portal.

### 4. Déploiement de l'Ops-Core (Layer 7 & Observabilité)
1. Créez un conteneur LXC Debian 12 (Privilégié, Nesting activé, 2 Go RAM, IP `10.0.0.5`).
2. Installez Docker Engine.
3. Copiez le dossier `ops-core/` dans `/opt/`.
4. Exécutez : `docker compose up -d`.
5. Configurez le DNS des clients DHCP pfSense pour pointer vers `10.0.0.5`.

---

## 👨‍💻 Auteur
Shalomé Ambinintsoa Ratsimbazafy
*Étudiant en Informatique Générale à l'École Nationale d'Informatique (ENI) - Fianarantsoa.*  
Ce projet a été réalisé dans le cadre de mon stage de seconde année en licence professionelle au sein de Meteo Madagascar.