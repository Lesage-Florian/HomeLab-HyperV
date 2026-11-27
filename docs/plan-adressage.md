[← Retour à l'accueil](../README.md)

# 🌐 Plan d'Adressage IP & Matrice de Flux

## 1. Découpage des VLANs

Le réseau est segmenté en 5 VLANs portés par des sous-interfaces **802.1Q** sur `LAB-FW` (OPNsense), qui joue le rôle de passerelle par défaut pour chaque segment et de serveur DHCP pour les VLANs utilisateurs.

| VLAN | Sous-réseau | IP Statiques / Plage DHCP | Passerelle | Broadcast |
| :--- | :--- | :--- | :--- | :--- |
| **10 — `VLAN-DIRECTION`**<br>*Postes Direction* | `192.168.10.0/24` | **DHCP** : `192.168.10.50` – `192.168.10.200` | `192.168.10.254` | `192.168.10.255` |
| **20 — `VLAN-RH`**<br>*Postes RH* | `192.168.20.0/24` | **DHCP** : `192.168.20.50` – `192.168.20.200` | `192.168.20.254` | `192.168.20.255` |
| **30 — `VLAN-ADMIN`**<br>*Poste Administrateur* | `192.168.30.0/24` | **Statique** : `192.168.30.10` (`LAB-PC-ADMIN`) | `192.168.30.254` | `192.168.30.255` |
| **40 — `VLAN-SUPERVISION`**<br>*Supervision* | `10.10.40.0/28` | **Statique** : `10.10.40.1` (`LAB-SUPERVISION`) | `10.10.40.14` | `10.10.40.15` |
| **100 — `VLAN-SRV`**<br>*Serveurs (`AD`, `FILE`, `BACKUP`)* | `10.10.100.0/24` | **Statiques** :<br>`10.10.100.1` (`LAB-SRV-AD`)<br>`10.10.100.2` (`LAB-SRV-FILE`)<br>`10.10.100.3` (`LAB-SRV-BACKUP`) | `10.10.100.254` | `10.10.100.255` |

> [!NOTE]
> Sur `VLAN-SUPERVISION` (`10.10.40.0/28`), le masque `/28` limite volontairement le sous-réseau à **14 adresses utilisables** (`10.10.40.1` à `10.10.40.14`) afin de réduire la surface d'exposition du segment de métrologie.

---

## 2. Matrice de Flux Inter-VLAN (OPNsense)

Par défaut, le trafic inter-VLAN est bloqué (*Deny All*). Les flux suivants sont explicitement autorisés dans **OPNsense** :

| Source | Destination | Ports / Protocole | Usage | Statut |
| :--- | :--- | :--- | :--- | :---: |
| `VLAN 10 / 20 / 30` | `LAB-SRV-AD` (`10.10.100.1`) | `UDP 53` | Résolution DNS interne | Déployé |
| `VLAN 10 / 20 / 30` | `LAB-SRV-AD` (`10.10.100.1`) | `TCP/UDP 88, 389`<br>`TCP 445` | Kerberos, LDAP, accès SYSVOL / NETLOGON | Déployé |
| `LAB-SUPERVISION` (`10.10.40.1`) | `LAB-FW` & `LAB-SRV-AD` | `UDP 161` | Collecte SNMP v2c (Poller Centreon) | Déployé |
| `VLAN 10 / 20` | `LAB-SRV-FILE` (`10.10.100.2`) | `TCP 445` | Accès aux partages réseau SMB | À finaliser |
| `VLAN 30` (`ADMIN`) | Tous les VLANs | `TCP 3389, 22, 443` | Administration (RDP, SSH, HTTPS) depuis `LAB-PC-ADMIN` | À durcir |
