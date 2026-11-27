[← Retour à l'accueil](../README.md)

# 📊 Supervision & Métrologie (Centreon)

## 1. Architecture de Supervision

- **Serveur** : `LAB-SUPERVISION` sous **AlmaLinux 9** (`10.10.40.1/28`, VLAN 40).
- **Protocole de collecte** : **SNMP v2c** (port `UDP 161`) autorisé à travers OPNsense depuis le Poller Centreon vers les équipements cibles.
- **Durcissement SNMP** : service SNMP déployé et restreint sur `LAB-SRV-AD` (Windows Server) et `LAB-FW` (OPNsense).

---

## 2. Sondes Actives & Tableau de Bord

Le tableau de bord de synthèse **« Globale View Infrastructure LAB »** regroupe :

- **Grille d'état globale** des hôtes supervisés (`LAB-FW`, `LAB-SRV-AD`, `LAB-SUPERVISION`).
- **Graphiques de métriques réseau** (vues **1 heure** et **24 heures**) sur l'ensemble des 5 interfaces VLAN d'OPNsense (`Traffic_VLAN_*`).
- **Jauges et compteurs système** pour `LAB-SRV-AD` :
  - Charge CPU (`total_cpu_avg`)
  - Utilisation RAM (`Memory`)
  - Espace disque (`C:`)

> [!TIP]
> La chaîne de collecte et la génération des courbes **RRDtool** ont été validées par injection de trafic inter-VLAN.

---

## 3. Évolutions & MCO (Maintien en Conditions Opérationnelles)

### Alerting & Notifications
- ⏳ Configurer un relais **Postfix / SMTP** sur `LAB-SUPERVISION` pour l'envoi d'e-mails d'alerte.
- ⏳ Créer les contacts et groupes de contacts (technicien / administrateur).
- ⏳ Définir les politiques de notification (ex. disque `C:` > 90 %, pare-feu `Down` > 2 minutes).
- ⏳ *(Optionnel)* Ajouter un webhook vers Discord, Slack ou Microsoft Teams pour les alertes critiques.

### Supervision Applicative Active Directory
- ⏳ Superviser les services Windows critiques de l'AD (`NTDS`, `DNS`, `Netlogon`).
- ⏳ Ajouter une sonde de résolution DNS (`check_dns`) sur `10.10.100.1`.
- ⏳ Ajouter des contrôles de disponibilité sur les ports TCP critiques : **LDAP (`389`)** et **Kerberos (`88`)**.

### Exploitation & MCO
- ⏳ Mettre en place une tâche `cron` sauvegardant les bases MariaDB (`centreon` et `centreon_storage`) ainsi que `/etc/centreon*`.
- ⏳ Ajuster le seuil *Warning* sur la RAM de `LAB-SRV-AD` (actuellement en alerte orange) afin d'éviter la fatigue d'alerte.
