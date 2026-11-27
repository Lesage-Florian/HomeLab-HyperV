[← Retour à l'accueil](../README.md)

# 💾 Serveur de Fichiers & Stratégie de Sauvegarde 3-2-1

## 1. Serveur de Fichiers (`LAB-SRV-FILE`)

- **Hôte** : `LAB-SRV-FILE` (`10.10.100.2/24`, VLAN 100), sous **Windows Server 2025 Standard**.
- **Architecture stockage** : séparation entre le volume système (`C:`) et le volume dédié aux données partagées.
- **Modèle de droits (Isolation Couche 7)** :
  - Partages distincts par département métier : `RH`, `Direction`, `Commun`.
  - Application des permissions **NTFS** et des droits de partage via des **groupes de sécurité Active Directory** rattachés aux Unités d'Organisation (`OU RH`, `OU Direction`).

---

## 2. Stratégie de Sauvegarde 3-2-1 (`LAB-SRV-BACKUP`)

L'architecture de sauvegarde repose sur `LAB-SRV-BACKUP` (`10.10.100.3/24`, VLAN 100) avec **Veeam Backup & Replication** selon la règle **3-2-1** :

> [!IMPORTANT]
> - **3 copies des données** : données de production sur les serveurs + sauvegarde locale + copie externalisée.
> - **2 supports différents** : disques virtuels de production + stockage dédié / NAS local.
> - **1 copie hors site** : externalisation cloud ciblée vers **AWS Glacier** pour l'archivage long terme et la reprise après sinistre.

### Actions restantes
- ⏳ Finaliser la configuration des volumes et partages SMB/NTFS sur `LAB-SRV-FILE`.
- ⏳ Installer et configurer **Veeam Backup & Replication** sur `LAB-SRV-BACKUP`.
- ⏳ Réaliser des tests de sauvegarde et de restauration (fichier unitaire + VM complète) pour valider les RPO/RTO.
