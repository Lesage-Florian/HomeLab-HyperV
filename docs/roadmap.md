[← Retour à l'accueil](../README.md)

# 🗺️ Feuille de route

## 1. Travaux réalisés

- **Socle Hyper-V & Réseau** : activation d'Hyper-V sous Windows 11, création de `vSwitch-LAB-INTERNE`, déploiement d'OPNsense (`LAB-FW`) avec routage inter-VLAN 802.1Q, DHCP sur les VLANs utilisateurs et règles de filtrage AD/SNMP.
- **Annuaire & Identités** : déploiement de `LAB-SRV-AD` (`10.10.100.1`) avec **AD DS** et **DNS**, forêt et domaine opérationnels, structure d'OU prévue pour `RH` et `Direction`.
- **Supervision** : déploiement de `LAB-SUPERVISION` (`10.10.40.1`) sous **AlmaLinux 9** avec **Centreon**, collecte **SNMP v2c** sur `LAB-FW` et `LAB-SRV-AD`, tableau de bord *« Globale View Infrastructure LAB »* validé par injection de trafic inter-VLAN.

---

## 2. Prochaines étapes techniques

### Phase A — Services & Sécurité Active Directory
- [ ] Finaliser la création des groupes de sécurité et l'application des **GPO** métier (mappage de lecteurs, restrictions, durcissement postes `DIRECTION`, `RH`, `ADMIN`).
- [ ] Finaliser `LAB-SRV-FILE` (`10.10.100.2`) avec séparation des volumes et partages SMB/NTFS (`RH`, `Direction`, `Commun`).

### Phase B — Sauvegarde & Résilience
- [ ] Déployer **Veeam Backup & Replication** sur `LAB-SRV-BACKUP` (`10.10.100.3`).
- [ ] Valider la stratégie **3-2-1** (dépôt local/NAS + réplication ciblée **AWS Glacier**) et documenter un test de restauration.

### Phase C — Supervision avancée & MCO
- [ ] Mettre en place l'alerting Centreon (Postfix/SMTP, contacts, seuils, webhook optionnel).
- [ ] Ajouter les sondes applicatives AD (`NTDS`, `DNS`, `Netlogon`, `check_dns`, ports `389`/`88`).
- [ ] Automatiser la sauvegarde de Centreon (`cron` MariaDB + `/etc/centreon*`) et ajuster le seuil RAM de `LAB-SRV-AD`.

---

## 3. Documentation & Preuves de fonctionnement à produire

- [ ] **Schéma d'architecture de supervision** : intégration des VLANs, du rôle d'OPNsense et du flux SNMP (`UDP 161`) de Centreon vers les cibles.
- [ ] **Matrice de supervision** : tableau récapitulatif (équipement cible IP/OS, sondes associées, seuils *Warning / Critical* et justifications techniques).
- [ ] **Preuves de fonctionnement** : captures d'écran du Dashboard Centreon incluant le pic de charge simulé.
- [ ] **Exports de configuration sanitisés** : sauvegarde XML d'OPNsense et scripts PowerShell d'automatisation AD/GPO.
