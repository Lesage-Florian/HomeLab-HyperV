# 🏢 Homelab Infrastructure d'Entreprise — Hyper-V, OPNsense, AD & Centreon

![Hyper-V](https://img.shields.io/badge/Hyperviseur-Hyper--V-0078D4?style=flat-square&logo=windows&logoColor=white)
![OPNsense](https://img.shields.io/badge/Pare--feu-OPNsense-D94F00?style=flat-square)
![Windows Server](https://img.shields.io/badge/Active%20Directory-Windows%20Server%202025-0078D6?style=flat-square&logo=windows&logoColor=white)
![Centreon](https://img.shields.io/badge/Supervision-Centreon%20%2F%20AlmaLinux%209-81B900?style=flat-square)
![Statut](https://img.shields.io/badge/Statut-En%20cours%20de%20d%C3%A9ploiement-orange?style=flat-square)

> **Projet Portfolio — Administrateur d'Infrastructures Sécurisées**  
> Maquette virtualisée sous **Windows 11 + Hyper-V (Type 1)** reproduisant l'infrastructure système et réseau segmentée d'une PME : routage inter-VLAN 802.1Q, filtrage pare-feu centralisé, annuaire Active Directory, serveur de fichiers, stratégie de sauvegarde 3-2-1 et supervision centralisée.

---

## 🎯 Objectifs du projet

Genèse du projet — Ce homelab s'inscrit dans le prolongement direct du projet fil rouge réalisé durant ma formation chez Liora. Ce dernier portait sur l'audit complet et la conception d'une nouvelle infrastructure pour une entreprise spécialisée dans l'analyse des fonds marins, engagée dans une phase de forte croissance à la suite d'une levée de fonds.

L'exercice initial s'arrêtant à la phase d'audit, d'architecture cible et de restitution documentaire (sans déploiement technique), j'ai créé ce homelab pour pousser la démarche jusqu'à l'implémentation concrète. Afin de respecter les capacités matérielles de ma machine hôte tout en restant fidèle aux exigences du projet, j'en ai reproduit les principes clés à échelle réduite.

Cet environnement permet ainsi de déployer, sécuriser, superviser et documenter en conditions quasi réelles les briques fondamentales d'un système d'information d'entreprise :

- **Segmentation réseau & Sécurité (Couches 2/3)** : isolation par VLANs 802.1Q via un commutateur virtuel dédié (`vSwitch-LAB-INTERNE`) et filtrage inter-VLAN centralisé par **OPNsense** (`LAB-FW`).
- **Services d'Infrastructure & Isolation système (Couche 7)** : annuaire **Active Directory (AD DS)**, résolution **DNS**, découpage par Unités d'Organisation (Direction, RH) et permissions NTFS/SMB.
- **Supervision & Métrologie** : collecte **SNMP v2c** centralisée avec **Centreon** sous **AlmaLinux 9** et tableau de bord de synthèse (*Globale View Infrastructure LAB*).
- **Résilience & Sauvegarde** : stratégie **3-2-1** combinant sauvegarde locale (**Veeam Backup & Replication**) et externalisation cloud ciblée (**AWS Glacier**).

---

## 🗺️ Architecture Réseau

```mermaid
flowchart LR
    WAN(["Réseau Hôte / WAN"]) --> FW["LAB-FW (OPNsense)<br/>Routage 802.1Q · Filtrage · DHCP"]
    FW ===|"Trunk 802.1Q<br/>VLAN 10, 20, 30, 40, 100"| VSW{{"vSwitch-LAB-INTERNE<br/>(Hyper-V Interne)"}}

    subgraph VLANS["Segments Réseau (VLANs)"]
        direction TB
        V10["<b>VLAN 10 — DIRECTION</b> · 192.168.10.0/24<br/>LAB-PC-DIRECTION (Win 11 · DHCP)"]
        V20["<b>VLAN 20 — RH</b> · 192.168.20.0/24<br/>LAB-PC-RH (Win 11 · DHCP)"]
        V30["<b>VLAN 30 — ADMIN</b> · 192.168.30.0/24<br/>LAB-PC-ADMIN (Win 11 · .10)"]
        V40["<b>VLAN 40 — SUPERVISION</b> · 10.10.40.0/28<br/>LAB-SUPERVISION (Centreon · 10.10.40.1)"]
        V100["<b>VLAN 100 — SERVEURS</b> · 10.10.100.0/24<br/>LAB-SRV-AD (.1) · LAB-SRV-FILE (.2) · LAB-SRV-BACKUP (.3)"]
    end

    VSW --- V10
    VSW --- V20
    VSW --- V30
    VSW --- V40
    VSW --- V100

    V40 -. "SNMP v2c (UDP 161)" .-> FW
    V40 -. "SNMP v2c (UDP 161)" .-> V100
    V10 -. "DNS 53 · Kerberos 88 · LDAP 389 · SMB 445" .-> V100
    V20 -. "DNS 53 · Kerberos 88 · LDAP 389 · SMB 445" .-> V100
```

---

## 🌐 Plan d'Adressage IP (VLANs)

| VLAN | Sous-réseau | IP Statiques / Plage DHCP | Passerelle | Broadcast |
| :--- | :--- | :--- | :--- | :--- |
| **10 — `VLAN-DIRECTION`**<br>*Postes Direction* | `192.168.10.0/24` | **DHCP** : `.50` – `.200` | `192.168.10.254` | `192.168.10.255` |
| **20 — `VLAN-RH`**<br>*Postes RH* | `192.168.20.0/24` | **DHCP** : `.50` – `.200` | `192.168.20.254` | `192.168.20.255` |
| **30 — `VLAN-ADMIN`**<br>*Poste Admin* | `192.168.30.0/24` | **Statique** : `192.168.30.10` (`LAB-PC-ADMIN`) | `192.168.30.254` | `192.168.30.255` |
| **40 — `VLAN-SUPERVISION`**<br>*Supervision* | `10.10.40.0/28` | **Statique** : `10.10.40.1` (`LAB-SUPERVISION`) | `10.10.40.14` | `10.10.40.15` |
| **100 — `VLAN-SRV`**<br>*Serveurs* | `10.10.100.0/24` | **Statiques** :<br>`10.10.100.1` (`LAB-SRV-AD`)<br>`10.10.100.2` (`LAB-SRV-FILE`)<br>`10.10.100.3` (`LAB-SRV-BACKUP`) | `10.10.100.254` | `10.10.100.255` |

> [!NOTE]
> La matrice complète des règles de filtrage inter-VLAN dans OPNsense est détaillée dans [docs/plan-adressage.md](docs/plan-adressage.md).

---

## 💻 Inventaire & Dimensionnement des VMs

| VM Hyper-V | OS | Rôle principal | VLAN | RAM (Type) |
| :--- | :--- | :--- | :---: | :--- |
| `LAB-FW` | OPNsense | Pare-feu, routage 802.1Q, DHCP | Trunk | 1024 – 2048 Mo *(Fixe)* |
| `LAB-SRV-AD` | Windows Server 2025 | Contrôleur de domaine (`AD DS`, `DNS`) | 100 | 2048 – 4096 Mo *(Dyn.)* |
| `LAB-SRV-FILE` | Windows Server 2025 | Serveur de fichiers (`SMB` / `NTFS`) | 100 | 2048 – 4096 Mo *(Dyn.)* |
| `LAB-SRV-BACKUP` | Windows / Linux | Sauvegarde (`Veeam` 3-2-1) | 100 | 2048 – 4096 Mo *(Dyn.)* |
| `LAB-SUPERVISION` | AlmaLinux 9 | Métrologie & alerting (`Centreon`) | 40 | 2048 – 4096 Mo *(Fixe)* |
| `LAB-PC-DIRECTION` | Windows 11 Pro | Poste client — Direction | 10 | 2048 – 4096 Mo *(Dyn.)* |
| `LAB-PC-RH` | Windows 11 Pro | Poste client — Ressources Humaines | 20 | 2048 – 4096 Mo *(Dyn.)* |
| `LAB-PC-ADMIN` | Windows 11 Pro | Poste d'administration dédié | 30 | 2048 – 4096 Mo *(Dyn.)* |

---

## ✅ État d'avancement & Feuille de route

### 1. Socle Réseau & Hyperviseur
- ✅ Activation d'Hyper-V (Windows 11) et création du commutateur virtuel `vSwitch-LAB-INTERNE`
- ✅ Déploiement d'**OPNsense** (`LAB-FW`) : sous-interfaces 802.1Q sur les 5 VLANs, passerelles et DHCP clients
- ✅ Ouverture sélective des flux inter-VLAN vers l'AD (`UDP 53`, `Kerberos 88`, `LDAP 389`, `SMB 445`) et SNMP (`UDP 161`)
- ⏳ Export et versionnement de la configuration XML OPNsense (sanitisée)

### 2. Services d'Infrastructure & Domaine
- ✅ Déploiement de `LAB-SRV-AD` (`10.10.100.1`) avec rôles **AD DS** et **DNS**
- ✅ Conception de la structure d'Unités d'Organisation (OU `RH`, `Direction`)
- ⏳ Application des **GPO** métier et durcissement des postes clients (`DIRECTION`, `RH`, `ADMIN`)

### 3. Fichiers & Sauvegarde (Stratégie 3-2-1)
- ✅ Validation du modèle de droits (partages `RH`, `Direction`, `Commun` + permissions NTFS par groupes AD)
- ✅ Validation de la stratégie de sauvegarde **3-2-1** (local/NAS + cible cloud AWS Glacier)
- ⏳ Déploiement complet de `LAB-SRV-FILE` (volumes dédiés, partages SMB/NTFS)
- ⏳ Déploiement et tests de restauration sur `LAB-SRV-BACKUP` (**Veeam Backup & Replication**)

### 4. Supervision & MCO (Centreon)
- ✅ Déploiement de `LAB-SUPERVISION` (`10.10.40.1`) sous AlmaLinux 9 et sécurisation de la collecte **SNMP v2c**
- ✅ Sondes actives : disponibilité hôtes (`FW`, `AD-SRV`, `Centreon-SRV`), CPU/RAM/Disque `C:` de l'AD, trafic des 5 VLANs OPNsense
- ✅ Création du tableau de bord *« Globale View Infrastructure LAB »* et validation par injection de trafic inter-VLAN (RRDtool)
- ⏳ Mise en place de l'**alerting** (relais SMTP/Postfix, contacts, politique de notification, option webhook Discord/Slack/Teams)
- ⏳ Ajout de la **supervision applicative AD** (services `NTDS`/`DNS`/`Netlogon`, sonde `check_dns`, ports TCP `389`/`88`)
- ⏳ Automatisation de la **sauvegarde Centreon** (cron MariaDB `centreon` + `centreon_storage` et `/etc/centreon*`)
- ⏳ Ajustement du seuil *Warning* RAM de `LAB-SRV-AD` pour supprimer l'alerte orange permanente

---

## 📚 Documentation détaillée

| Document | Contenu |
| :--- | :--- |
| [Architecture & Dimensionnement](docs/architecture.md) | Prérequis Hyper-V, `vSwitch`, dimensionnement VMs, isolation L2/L3 vs L7 |
| [Plan d'Adressage & Flux](docs/plan-adressage.md) | Découpage des 5 VLANs et matrice de filtrage OPNsense |
| [Supervision Centreon & MCO](docs/supervision.md) | Collecte SNMP v2c, sondes actives, alerting et sauvegarde MariaDB |
| [Fichiers & Sauvegarde 3-2-1](docs/sauvegarde.md) | Modèle de droits `LAB-SRV-FILE` et stratégie Veeam + AWS Glacier |
| [Feuille de route](docs/roadmap.md) | Bilan des réalisations, prochaines phases et documentation à produire |
