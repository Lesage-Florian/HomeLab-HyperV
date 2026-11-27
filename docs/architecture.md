[← Retour à l'accueil](../README.md)

# 🏗️ Architecture & Dimensionnement Hyper-V

## 1. Prérequis Matériels & Hyperviseur

L'environnement repose sur **Hyper-V** (hyperviseur de type 1 intégré nativement à Windows 11 Pro / Enterprise / Education), dimensionné pour exécuter simultanément l'ensemble de la topologie :

- **Processeur** : Multicœur récent supportant la virtualisation matérielle (`VT-x` / `AMD-V` activée dans le BIOS).
- **Mémoire RAM** : 16 Go minimum (32 Go recommandés pour exécuter toutes les VMs simultanément sans contrainte).
- **Stockage** : SSD NVMe avec au moins 250 Go d'espace libre.

### Configuration réseau virtuelle

Un commutateur virtuel de type **Interne** nommé `vSwitch-LAB-INTERNE` isole les communications inter-VMs sur l'hôte Hyper-V. L'interface LAN/Trunk de `LAB-FW` (OPNsense) y est raccordée et porte les sous-interfaces taguées **802.1Q** pour chacun des 5 VLANs.

---

## 2. Dimensionnement des Machines Virtuelles

> [!TIP]
> Pour optimiser la consommation mémoire sur l'hôte, les postes et serveurs Windows utilisent la **RAM Dynamique**, tandis que les appliances critiques (`LAB-FW`, `LAB-SUPERVISION`) utilisent une **RAM Fixe**.

| VM Hyper-V | OS | Rôle dans l'architecture | RAM (Type) |
| :--- | :--- | :--- | :--- |
| `LAB-FW` | OPNsense | Pare-feu, routage inter-VLAN 802.1Q, DHCP | 1024 – 2048 Mo *(Fixe)* |
| `LAB-SRV-AD` | Windows Server 2025 | Contrôleur de domaine (`AD DS`, `DNS`) | 2048 – 4096 Mo *(Dynamique)* |
| `LAB-SRV-FILE` | Windows Server 2025 | Serveur de fichiers, partages `SMB` / `NTFS` | 2048 – 4096 Mo *(Dynamique)* |
| `LAB-SRV-BACKUP` | Windows / Linux | Sauvegarde (`Veeam Backup & Replication`) | 2048 – 4096 Mo *(Dynamique)* |
| `LAB-SUPERVISION` | AlmaLinux 9 | Serveur de métrologie & alerting (`Centreon`) | 2048 – 4096 Mo *(Fixe)* |
| `LAB-PC-DIRECTION` | Windows 11 Pro | Poste utilisateur — Service Direction | 2048 – 4096 Mo *(Dynamique)* |
| `LAB-PC-RH` | Windows 11 Pro | Poste utilisateur — Service RH | 2048 – 4096 Mo *(Dynamique)* |
| `LAB-PC-ADMIN` | Windows 11 Pro | Poste administrateur dédié | 2048 – 4096 Mo *(Dynamique)* |

---

## 3. Principes de Sécurité & Bonnes Pratiques

### Isolation Réseau vs Isolation Système

- **Isolation Réseau (Couches 2 & 3)** : assurée au niveau de l'infrastructure par le taggage **VLAN ID (802.1Q)** sur les cartes réseau virtuelles Hyper-V et l'application de règles de filtrage strictes dans **OPNsense**. Elle empêche tout trafic non autorisé de transiter d'un sous-réseau à un autre.
- **Isolation Système (Couche 7 / Application)** : assurée par la gestion des identités dans **Active Directory**. L'accès aux ressources (dossiers partagés, imprimantes) est régi par les droits **NTFS** et les autorisations de partage attribuées aux groupes de sécurité de domaine, indépendamment de l'emplacement réseau.

### Gestion des Points de Contrôle (Snapshots)

> [!IMPORTANT]
> Des points de contrôle Hyper-V sont créés systématiquement à deux moments clés :
> 1. Juste après l'installation de base de l'OS (avant personnalisation).
> 2. Avant toute opération critique (promotion en contrôleur de domaine, installation de Centreon, refonte de règles pare-feu).
