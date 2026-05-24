# Architecture Decision Records (ADR)

Ce dossier contient les décisions architecturales de la plateforme de facturation électronique marocaine, documentées sous forme d'ADR (Architecture Decision Records).

## Pourquoi des ADR

Les ADR tracent le contexte, les options considérées, et la justification des décisions architecturales significatives. Ils sont particulièrement critiques dans un contexte réglementé car ils permettent de démontrer lors d'un audit que les choix techniques ont été délibérés et justifiés au regard des contraintes légales.

## Format des ADR

Chaque ADR suit la structure suivante :

```markdown
# ADR-XXXX — Titre court et descriptif

**Date** : YYYY-MM-DD
**Statut** : Proposé | Accepté | Déprécié | Remplacé par ADR-YYYY
**Auteur** : Nom ou agent
**Réviseurs** : Agents ou personnes ayant validé

## Contexte

Description du problème ou de la contrainte qui nécessite une décision.
Inclure les contraintes réglementaires applicables (DGI, DGSSI, CNDP, ISO 27001).

## Options considérées

1. **Option A** — Description, avantages, inconvénients
2. **Option B** — Description, avantages, inconvénients
3. **Option C** — Description, avantages, inconvénients

## Décision

Option retenue et justification détaillée.

## Conséquences

- Impact positif attendu
- Contraintes introduites
- Décisions futures conditionnées par cet ADR

## Références réglementaires

- [Référence à l'exigence DGI, DGSSI, CNDP, ou contrôle ISO 27001 concerné]
```

## Index des ADR

> Les ADR sont créés au fil des décisions architecturales. Chaque ADR est nommé `ADR-XXXX-titre-court.md`.

## Domaines couverts

Les ADR sont attendus pour toute décision touchant à :
- Format et structure des factures électroniques
- Algorithmes et mécanismes de signature
- Architecture d'intégration avec les systèmes DGI
- Choix de bibliothèques cryptographiques
- Architecture de gestion des clés (HSM, KMS)
- Stratégie d'archivage et de rétention
- Modèle de données contenant des PII
- Frontières de confiance et modèle d'authentification
