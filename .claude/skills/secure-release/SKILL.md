# Purpose

Valider qu'une release candidate est sécurisée, conforme, et prête pour un déploiement en production sur la plateforme de facturation électronique marocaine. Gate de sécurité et de conformité obligatoire avant chaque déploiement production.

# Trigger Conditions

- Avant tout déploiement en environnement de production.
- Avant tout déploiement en environnement de staging recevant des données réelles.
- Après tout correctif de sécurité ou de conformité critique.
- Demande explicite de l'agent `security` ou du responsable release.

# Execution Steps

1. **Vérifier la complétude de la release** :
   - Tous les tests passent (unitaires, intégration, conformité DGI)
   - Aucun test de sécurité en échec
   - Changelog documenté avec références aux exigences réglementaires impactées
2. **Audit des dépendances** :
   - Aucune dépendance avec CVE critique ou haute non résolue
   - Bibliothèques cryptographiques approuvées DGSSI uniquement
   - Licences compatibles avec l'usage production
3. **Vérification de la configuration** :
   - Aucun secret exposé dans le code ou les configurations
   - TLS 1.3 configuré pour toutes les communications externes
   - Mode debug désactivé
   - Endpoints de diagnostic non exposés
4. **Validation de conformité** :
   - Revue `compliance-review` complétée et approuvée
   - Revue `pii-review` complétée pour les fonctionnalités traitant des données personnelles
   - Journaux d'audit activés et fonctionnels
   - Mécanismes de signature électronique validés
5. **Vérifier la procédure de rollback** :
   - Procédure de rollback documentée et testée
   - Point de restauration défini
   - Notification DGI planifiée si le changement impacte les interfaces de transmission
6. **Produire le rapport de release** — Synthèse des contrôles effectués, conformité établie, risques résiduels acceptés, approbations obtenues.

# Success Criteria

- Tous les contrôles de la checklist sont verts ou documentés avec justification.
- Les approbations des agents `compliance`, `security`, et `audit` sont obtenues.
- La procédure de rollback est testée et disponible.
- Le rapport de release est archivé dans `docs/control-evidence/`.

# Failure Handling

- CVE critique non résolue → blocage immédiat du déploiement + plan de remédiation sous 24h.
- Test de conformité DGI en échec → blocage + notification équipe réglementaire + analyse d'impact.
- Secret détecté → blocage + révocation immédiate + rotation + analyse d'exposition.
- Approbation manquante → déploiement refusé sans exception, même sous pression de délai.
