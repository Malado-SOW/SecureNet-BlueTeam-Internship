# Flux Réseau Critiques – SecureNet Solutions

## Méthodologie d'Analyse
Chaque flux est évalué selon :
1. Nécessité métier (justification)
2. Risque sécurité (impact si compromis)
3. Mesures de détection (comment le SIEM le surveille)
4. Mesures de prévention (comment le firewall le restreint)

---

## Flux #1 : Authentification Active Directory
### Source → Destination : VBoxNet-CLI → WIN-DC01 (195.221.0.35)
### Ports : TCP/88 (Kerberos), TCP/389 (LDAP), TCP/445 (SMB)
### Protocoles : Kerberos, LDAP, SMB

#### Justification Métier
- Les postes clients doivent s'authentifier auprès du domaine pour accéder aux ressources.
- LDAP pour la résolution des objets AD (utilisateurs, groupes).
- SMB pour l'accès aux partages réseau et GPO.

#### Risques Sécurité
- 🔴 Kerberos : attaques par force brute, ticket forging (Golden Ticket), Kerberoasting
- 🔴 LDAP : enumeration d'utilisateurs, injection LDAP si application vulnérable
- 🔴 SMB : exploits EternalBlue, relais d'authentification (NTLM relay)

#### Détection (SIEM Wazuh)
```xml
<!-- Exemple de règle Wazuh pour détecter Kerberoasting -->
<rule id="100100" level="12">
  <if_sid>60900</if_sid>
  <match>event_id:4769</match>
  <field name="win.eventdata.failureCode">0x1</field>
  <description>Potential Kerberoasting attempt - TGS request with RC4 encryption</description>
  <mitre>
    <id>T1558.003</id> <!-- Kerberoasting -->
  </mitre>
</rule>
```

## Flux #2 : Résolution DNS interne
### Source → Destination : VBoxNet-CLI → VBoxNet-SRV
### Ports : TCP/53
### Protocoles : DNS

#### Justification Métier
- Permet aux machines clientes de résoudre les noms des services internes.
- Indispensable pour localiser les services du réseau interne.
- Facilite l’accès aux applications et serveurs via leurs noms.

#### Risques Sécurité
- 🔴 DNS tunneling pour exfiltration de données.
- 🔴 DNS spoofing pour rediriger vers des ressources malveillantes.
- 🔴 Enumeration DNS pour cartographier le réseau interne.

#### Détection (SIEM Wazuh)
```xml
<!-- Détection volume anormal de requêtes DNS -->
<rule id="100110" level="10">
  <if_sid>530</if_sid>
  <match>dns</match>
  <frequency>100</frequency>
  <timeframe>60</timeframe>
  <description>Possible abnormal DNS query activity</description>
  <mitre>
    <id>T1071.004</id>
  </mitre>
</rule>
```
## Flux #3 : Administration et supervision
### Source → Destination : VBoxNet-MGT → Tous
### Ports : TCP/22, TCP/443, TCP/1514
### Protocoles : SSH, HTTPS, Wazuh

#### Justification Métier
- Administration distante des serveurs via SSH.
- Accès aux interfaces web sécurisées des services.
- Supervision et collecte d’événements via l’agent Wazuh.

#### Risques Sécurité
- 🔴 Compromission du poste d’administration permettant le contrôle du SI.
- 🔴 Accès SSH non autorisé.
- 🔴 Accès aux consoles d’administration via HTTPS.

#### Détection (SIEM Wazuh)
```xml
<!-- Détection brute force SSH -->
<rule id="100120" level="12">
  <if_sid>5710</if_sid>
  <match>authentication failure</match>
  <frequency>5</frequency>
  <timeframe>60</timeframe>
  <description>Multiple SSH authentication failures detected</description>
  <mitre>
    <id>T1110</id>
  </mitre>
</rule>
```

## Flux #4 : Simulation d'attaques contrôlées
### Source → Destination : KALI-ATT01 → VBoxNet-SRV
### Ports : TCP/*
### Protocoles : Tests de sécurité contrôlés

#### Justification Métier
- Simulation d’attaques dans le cadre d’exercices Blue Team.
- Test des capacités de détection du SIEM.
- Validation de la sécurité du réseau interne.

#### Risques Sécurité
- 🔴 Exploitation de vulnérabilités si les tests ne sont pas contrôlés.
- 🔴 Risque d’interruption de service.
- 🔴 Possibilité de fausses alertes dans les outils de supervision.

#### Détection (SIEM Wazuh)
```xml
<!-- Détection scan réseau -->
<rule id="100130" level="10">
  <if_sid>5712</if_sid>
  <match>scan</match>
  <description>Possible network scan detected</description>
  <mitre>
    <id>T1046</id>
  </mitre>
</rule>
```