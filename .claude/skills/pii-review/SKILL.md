# Purpose

Identifier, classifier et évaluer les données à caractère personnel (PII) dans le code, les schémas de données et les flux de traitement. Vérifier la conformité au regard de la loi marocaine 09-08 (CNDP) et du GDPR pour les données de ressortissants européens.

# Trigger Conditions

- Toute PR créant ou modifiant un modèle de données, un schéma de base de données, ou un flux de traitement.
- Toute nouvelle intégration avec un système tiers pouvant recevoir des données personnelles.
- Toute fonctionnalité de log, d'export ou de reporting.
- Demande explicite de l'agent `privacy`.
- Avant tout déploiement en environnement de staging ou de production.

# Execution Steps

1. **Scanner les données traitées** — Identifier toutes les variables, champs, colonnes, et payloads manipulés par le changement.
2. **Classifier les données** :
   - PII directe : nom, prénom, NIF, CIN, adresse, email, téléphone
   - PII indirecte : données pouvant identifier une personne par combinaison
   - Données sensibles : données bancaires, données fiscales, données de santé
   - Données non personnelles : données purement techniques ou anonymisées
3. **Vérifier la base légale** — Pour chaque traitement de PII : base légale documentée (consentement, obligation légale, intérêt légitime) ?
4. **Vérifier la minimisation** — Les données collectées sont-elles strictement nécessaires à la finalité déclarée ?
5. **Vérifier les frontières PII** — Les modules manipulant des PII sont-ils isolés et annotés `@pii-boundary` ?
6. **Vérifier la pseudonymisation** — Les PII sont-elles pseudonymisées dans les logs, les environnements non-prod, les exports analytiques ?
7. **Vérifier les durées de rétention** — Chaque catégorie de PII a-t-elle une durée de rétention définie et appliquée ?
8. **Produire le rapport PII** — Inventaire des PII, classification, conformité CNDP/GDPR, risques identifiés.

# Success Criteria

- Aucune PII non classifiée dans le périmètre analysé.
- Chaque traitement de PII a une base légale documentée dans le registre des traitements.
- Zéro PII réelle dans les fixtures de test, les logs applicatifs, ou les environnements non-production.
- Les frontières PII sont annotées et respectées dans l'architecture.

# Failure Handling

- PII détectée dans les logs → blocage immédiat + demande de correction avant merge.
- PII réelle dans les données de test → blocage + purge + génération de données synthétiques équivalentes.
- Traitement sans base légale → blocage + escalade à l'agent `privacy` + équipe juridique.
- PII non classifiée dans un schéma de données → blocage + classification obligatoire avant approbation.
