# J5 – Plan d'Adressage IP et Documentation Réseau

## Fichiers Produits
- [x] `01_Documentation_Generale/PLAN_ADRESSAGE_IP.md` (tableau complet)
- [x] `01_Documentation_Generale/FLUX_RESEAU_CRITIQUES.md` (analyse sécurité par flux)
- [x] Mise à jour du README.md avec lien vers la documentation réseau

## Validations Sécurité
- [ ] Aucun chevauchement de plages IP
- [ ] Passerelles cohérentes avec l'architecture pfSense
- [ ] Réservations DHCP documentées pour postes critiques
- [ ] Flux inter-segments justifiés et minimisés

## Prochaines Étapes
- [ ] Utiliser ce plan pour configurer pfSense (Sprint 4)
- [ ] Configurer les IP statiques des serveurs lors de leur installation
- [ ] Documenter toute déviation au plan dans un fichier CHANGELOG_RESEAU.md