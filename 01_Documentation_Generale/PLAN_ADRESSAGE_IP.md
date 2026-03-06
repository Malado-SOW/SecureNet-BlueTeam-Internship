# Plan d'Adressage IP – SecureNet Solutions
## Plage Globale : 195.221.0.0/24
## Masque : 255.255.255.0 | Passerelle Générale : pfSense-FW01

---

## 📡 Sous-Réseau 1 : MANAGEMENT (MGT)
### Plage : 195.221.0.0/27 (32 adresses, 30 utilisables)
### Passerelle : 195.221.0.1 (pfSense LAN_MGT)

| Hôte | IP Statique | Rôle | OS | Notes Sécurité |
|------|------------|------|-----|----------------|
| pfSense-FW01 | 195.221.0.1 | Passerelle / Firewall | pfSense | Interface management, accès SSH restreint |
| LX-SIEM01 | 195.221.0.5 | SIEM Wazuh Manager | Ubuntu 22.04 | Accès Web HTTPS uniquement depuis IPs admin |
| [Réservé] | 195.221.0.2-4,6-30 | Extension future | - | Ne pas attribuer sans validation change management |

### Règles Firewall Associées
- ✅ Autorisé : pfSense → LX-SIEM01 : TCP/1514 (Wazuh agent registration)
- ✅ Autorisé : Admin Workstation (IP fixe) → LX-SIEM01 : TCP/443 (Dashboard)
- ❌ Bloqué : Toute autre source → MGT (principe du moindre privilège)

### Risques Sécurité
- 🔴 Exposition du dashboard SIEM : pourrait révéler des alertes à un attaquant
- 🔴 Accès SSH non restreint : brute force possible sur interface management
- ✅ Mitigation : Fail2ban sur SSH, authentification forte sur dashboard, logs centralisés

---

## 🖥️ Sous-Réseau 2 : SERVEURS (SRV)
### Plage : 195.221.0.32/27 (32 adresses, 30 utilisables)
### Passerelle : 195.221.0.33 (pfSense LAN_SRV)

| Hôte | IP Statique | Rôle | OS | Notes Sécurité |
|------|------------|------|-----|----------------|
| pfSense-FW01 | 195.221.0.33 | Passerelle segment SRV | pfSense | Routage inter-VLAN |
| WIN-DC01 | 195.221.0.35 | Active Directory, DNS, DHCP | Windows Server 2022 | DNS pointe vers lui-même (127.0.0.1) |
| LX-APP01 | 195.221.0.40 | Serveur applicatif Linux | Ubuntu 22.04 | Services : Apache, MySQL (à durcir) |
| [Réservé] | 195.221.0.34,36-39,41-62 | Extension future | - | Réserver IPs pour futurs serveurs |

### Règles Firewall Associées
- ✅ Autorisé : VBoxNet-CLI → WIN-DC01 : TCP/88,389,445 (Auth AD)
- ✅ Autorisé : VBoxNet-CLI → LX-APP01 : TCP/80,443 (Accès applicatif)
- ✅ Autorisé : Tous segments → WIN-DC01 : UDP/53 (DNS)
- ❌ Bloqué : VBoxNet-ATK → SRV (sauf règles de test explicites et temporaires)

### Risques Sécurité
- 🔴 WIN-DC01 exposé : cible prioritaire pour attaques AD (Kerberoasting, AS-REP Roasting)
- 🔴 LX-APP01 : risque d'injection SQL ou RCE si application non sécurisée
- ✅ Mitigation : GPO de durcissement AD, WAF applicatif, supervision logs critiques

---

## 💻 Sous-Réseau 3 : CLIENTS (CLI)
### Plage : 195.221.0.64/26 (64 adresses, 62 utilisables)
### Passerelle : 195.221.0.65 (pfSense LAN_CLI)

| Hôte | IP Attribution | Rôle | OS | Notes Sécurité |
|------|---------------|------|-----|----------------|
| pfSense-FW01 | 195.221.0.65 | Passerelle segment CLI | pfSense | DHCP relay si nécessaire |
| WIN-CLI01 | DHCP Réservé | Poste utilisateur 1 | Windows 11 Enterprise | Joint au domaine, BitLocker activé |
| WIN-CLI02 | DHCP Réservé | Poste utilisateur 2 | Windows 11 Enterprise | Même politique que CLI01 |
| [Pool DHCP] | 195.221.0.70-126 | Attribution dynamique | - | Lease time : 24h, réservations pour postes fixes |

### Configuration DHCP (pfSense)
Pool DHCP CLI :
Range : 195.221.0.70 - 195.221.0.126
Gateway : 195.221.0.65
DNS : 195.221.0.35 (WIN-DC01)
Domain : securenet.local
Lease Time : 86400 seconds (24h)

### Règles Firewall Associées
- ✅ Autorisé : CLI → SRV : ports AD et applicatifs (voir tableau SRV)
- ✅ Autorisé : CLI → WAN : TCP/80,443 (sortant uniquement, navigation web)
- ❌ Bloqué : CLI → MGT (isolation administration)
- ❌ Bloqué : CLI → CLI (empêche propagation latérale de malware)

### Risques Sécurité
- 🔴 Postes clients : vecteur d'entrée principal (phishing, drive-by download)
- 🔴 Propagation latérale : si un poste est compromis, risque de mouvement vers SRV
- ✅ Mitigation : EDR sur postes, restrictions firewall inter-CLI, supervision comportements anormaux

---

## ⚔️ Sous-Réseau 4 : ATTAQUE / RED TEAM (ATK)
### Plage : 195.221.0.128/27 (32 adresses, 30 utilisables)
### Passerelle : 195.221.0.129 (pfSense LAN_ATK)

| Hôte | IP Statique | Rôle | OS | Notes Sécurité |
|------|------------|------|-----|----------------|
| pfSense-FW01 | 195.221.0.129 | Passerelle segment ATK | pfSense | Logging renforcé sur cette interface |
| KALI-ATT01 | 195.221.0.130 | Machine de test d'intrusion | Kali Linux 2024 | Outils : nmap, metasploit, mimikatz, bloodhound |
| [Réservé] | 195.221.0.131-158 | Extension future | - | Pour futures machines d'attaque ou honeypots |

### Règles Firewall Associées (SPÉCIALES – Tests uniquement)
- ✅ Autorisé (temporaire) : KALI-ATT01 → SRV : ports de test (documentés dans chaque scénario)
- ✅ Autorisé : KALI-ATT01 → WAN : TCP/80,443 (téléchargement outils de test)
- ❌ Bloqué par défaut : ATK → MGT (isolation stricte de l'admin)
- ❌ Bloqué par défaut : ATK → CLI (empêche attaque directe des postes)

### Logging Renforcé pour ATK
Sur pfSense, activer pour l'interface LAN_ATK :
Log all blocked packets (voir ce qui est tenté)
Log all passed packets (voir ce qui réussit – pour analyse post-test)
Forward logs to SIEM : TCP/514 vers LX-SIEM01

### Risques Sécurité (par conception)
- 🔴 KALI contient des outils d'attaque : si fuite réseau, risque réel
- 🔴 Tests mal configurés : peuvent crasher des services (DoS involontaire)
- ✅ Mitigation : Isolation réseau stricte, snapshots avant chaque test, procédure de rollback

---

## 📊 Récapitulatif des Plages

| Segment | CIDR | Premier IP | Dernier IP | Passerelle | Usage Principal |
|---------|------|-----------|------------|------------|----------------|
| MGT | 195.221.0.0/27 | .1 | .30 | .1 | Administration infrastructure |
| SRV | 195.221.0.32/27 | .33 | .62 | .33 | Hébergement services critiques |
| CLI | 195.221.0.64/26 | .65 | .126 | .65 | Postes utilisateurs finaux |
| ATK | 195.221.0.128/27 | .129 | .158 | .129 | Tests d'intrusion contrôlés |
| RÉSERVE | 195.221.0.160/27 | .161 | .190 | - | Extension future |
| NON UTILISÉ | 195.221.0.192/26 | .193 | .254 | - | Buffer / sécurité |

