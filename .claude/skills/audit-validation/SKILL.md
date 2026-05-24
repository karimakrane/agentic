# Purpose

Vérifier l'intégrité, la complétude, et la conformité des journaux d'audit et des preuves de contrôle de la plateforme de facturation électronique marocaine. Préparer les dossiers d'audit pour les inspections réglementaires DGI, CNDP, et les certifications ISO 27001.

# Trigger Conditions

- Avant toute inspection réglementaire planifiée (DGI, CNDP, ISO 27001).
- À intervalles réguliers définis (mensuel pour les journaux d'audit, trimestriel pour les preuves de contrôle ISO 27001).
- Après tout incident de sécurité ou de conformité.
- Demande explicite de l'agent `audit`.
- Avant toute migration ou archivage de données fiscales.

# Execution Steps

1. **Inventorier les journaux d'audit** :
   - Identifier tous les journaux d'audit actifs et archivés
   - Vérifier la couverture : toutes les transactions fiscales sont-elles journalisées ?
   - Vérifier l'horodatage : synchronisation NTP, fuseau horaire cohérent
2. **Vérifier l'intégrité des journaux** :
   - Signatures cryptographiques des entrées de journal valides
   - Absence de lacunes (gaps) dans les séquences de journaux
   - Impossibilité de modification non détectée (immutabilité)
3. **Vérifier les durées de rétention** :
   - Journaux fiscaux : conformité aux durées légales marocaines
   - Journaux de sécurité : conformité ISO 27001 et DGSSI
   - Journaux de données personnelles : conformité CNDP (loi 09-08)
4. **Valider les preuves de contrôle ISO 27001** :
   - Vérifier que chaque contrôle de l'Annexe A déclaré applicable a une preuve associée dans `docs/control-evidence/`
   - Vérifier que les preuves sont datées, signées, et non expirées
5. **Identifier les lacunes** — Documenter chaque contrôle sans preuve ou avec preuve expirée.
6. **Produire le rapport d'audit** — État de chaque domaine, lacunes identifiées, plan de remédiation, niveau de préparation à l'inspection.

# Success Criteria

- Toutes les transactions fiscales des 12 derniers mois ont une entrée de journal d'audit vérifiable.
- Intégrité des journaux établie par vérification cryptographique.
- Toutes les preuves de contrôle ISO 27001 applicables sont présentes et à jour.
- Le rapport est archivé avec horodatage dans `docs/control-evidence/`.

# Failure Handling

- Lacune dans les journaux d'audit de transactions fiscales → incident critique + investigation immédiate + notification DGI si légalement requis.
- Signature de journal invalide → investigation forensique + gel des opérations de purge + escalade sécurité.
- Preuve de contrôle ISO 27001 manquante à J-30 d'un audit → alerte direction + mobilisation de l'équipe pour collecte d'urgence.
- Durée de rétention non respectée (données purgées prématurément) → incident de conformité + analyse d'impact légal.
