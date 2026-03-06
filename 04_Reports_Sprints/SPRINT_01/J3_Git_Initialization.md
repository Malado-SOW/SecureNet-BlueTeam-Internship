# J3 – Initialisation Git et Structure de Documentation

## Configuration Git
- Nom utilisateur : Malado-SOW
- Email : malado15sow03@gmail.com
- Version Git : 2.45.2.windows.1

## Structure du Repository
/01_Documentation_Generale # Architecture, politiques, plans IP
/02_Infrastructure_as_Code # Scripts de déploiement automatisé
/03_SIEM_Wazuh # Règles, décodeurs, dashboards personnalisés
/04_Reports_Sprints # Livrables hebdomadaires structurés
/05_Incident_Response # Playbooks et analyses post-incident

## Fichier .gitignore
- [ ] Exclusion des disques VM (.vdi, .vhd)
- [ ] Exclusion des secrets et credentials
- [ ] Exclusion des logs temporaires
- [ ] Exclusion du dossier ISO

## Premier Commit
- Hash : d79113e...
- Message : "feat: initialisation projet SecureNet..."
- Fichiers inclus : README.md, .gitignore, arborescence dossiers

## Prochaines Étapes
- [ ] Push vers remote (GitHub/GitLab)
- [ ] Création des fichiers de documentation par sprint