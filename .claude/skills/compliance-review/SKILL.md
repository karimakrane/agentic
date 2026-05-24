# Purpose

Évaluer la conformité réglementaire d'une PR, d'un module, ou d'un flux de données contre les exigences DGI, DGSSI, CNDP, ISO 27001 et GDPR applicables à la plateforme de facturation électronique marocaine.

# Trigger Conditions

- Toute PR modifiant : format de facture, flux de transmission DGI, mécanisme de signature, traitement de données personnelles, durées de rétention.
- Toute nouvelle fonctionnalité déclarée comme ayant un impact réglementaire.
- Demande explicite d'un agent `compliance` ou d'un reviewer humain.
- Revue périodique planifiée de la cartographie de conformité.

# Execution Steps

1. **Identifier le périmètre réglementaire** — Déterminer quelles réglementations s'appliquent au changement (DGI, DGSSI, CNDP, ISO 27001, GDPR).
2. **Lire les exigences applicables** — Consulter les documents dans `compliance/` correspondant au périmètre identifié.
3. **Analyser le changement** — Lire le code, les schémas, les configurations concernés. Identifier les exigences satisfaites et les écarts.
4. **Vérifier les points critiques** :
   - Format des factures conforme aux spécifications DGI en vigueur
   - Signature électronique avec certificat qualifié valide
   - Données personnelles traitées avec base légale documentée
   - Durées de rétention respectées
   - Journaux d'audit présents et conformes
   - Contrôles ISO 27001 de l'Annexe A couverts
5. **Produire le rapport** — Documenter : exigences vérifiées, conformité établie, écarts identifiés, recommandations, niveau de risque résiduel.
6. **Décision** — Approuver, approuver avec réserves documentées, ou bloquer avec justification réglementaire.

# Success Criteria

- Toutes les exigences réglementaires applicables sont évaluées et documentées.
- Chaque écart est associé à un risque quantifié et un plan de remédiation.
- Le rapport est tracé dans `docs/control-evidence/` avec horodatage.
- La décision est claire, justifiée, et référence les textes réglementaires applicables.

# Failure Handling

- Si les exigences réglementaires applicables sont ambiguës → documenter l'ambiguïté, escalader à l'équipe juridique, bloquer le changement en attendant clarification.
- Si des preuves de contrôle sont manquantes → bloquer le changement, requérir la collecte des preuves manquantes.
- Si une exigence DGI a évolué depuis la dernière revue → mettre à jour la cartographie, évaluer l'impact sur les contrôles existants, ouvrir un risque si nécessaire.
