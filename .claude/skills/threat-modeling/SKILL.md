# Purpose

Analyser les menaces de sécurité pesant sur un composant, une fonctionnalité ou un flux de données en appliquant la méthodologie STRIDE dans le contexte de la plateforme de facturation électronique marocaine. Produire un modèle de menaces documenté et des recommandations de contre-mesures.

# Trigger Conditions

- Toute nouvelle fonctionnalité exposant une interface externe (API, webhook, intégration DGI).
- Tout changement dans les mécanismes d'authentification, de signature, ou de chiffrement.
- Tout nouveau flux de données traversant une frontière de confiance.
- Demande explicite de l'agent `threat-modeling`.
- Revue annuelle du modèle de menaces global.

# Execution Steps

1. **Délimiter le périmètre** — Identifier les composants, flux de données, acteurs et frontières de confiance concernés.
2. **Construire le DFD** — Documenter le diagramme de flux de données (Data Flow Diagram) du périmètre analysé dans `threat-model/`.
3. **Appliquer STRIDE par élément** :
   - **Spoofing** — Usurpation d'identité fiscale, faux certificats, replay d'authentification
   - **Tampering** — Falsification de factures, altération des journaux d'audit, modification en transit
   - **Repudiation** — Déni de transaction fiscale, absence de non-répudiation sur les signatures
   - **Information Disclosure** — Fuite de données fiscales, exposition de NIF, accès non autorisé aux archives
   - **Denial of Service** — Indisponibilité de la transmission DGI, saturation des files de traitement
   - **Elevation of Privilege** — Accès non autorisé aux fonctions administratives, contournement d'autorisation
4. **Évaluer chaque menace** — Probabilité, impact (CVSS si applicable), contrôles existants, risque résiduel.
5. **Recommander des contre-mesures** — Associer chaque menace non couverte à un contrôle ISO 27001 de l'Annexe A ou une mesure technique spécifique.
6. **Mettre à jour `threat-model/`** — Documenter le DFD, le registre des menaces, et les décisions de traitement.

# Success Criteria

- Toutes les catégories STRIDE sont évaluées pour chaque élément du périmètre.
- Chaque menace de niveau élevé ou critique a un contrôle compensatoire documenté.
- Le modèle de menaces est mis à jour dans `threat-model/` avec horodatage.
- Les menaces résiduelles acceptées sont formellement approuvées et documentées.

# Failure Handling

- Menace critique sans contrôle compensatoire → blocage du déploiement + escalade à l'agent `security` + notification DGSSI si requis.
- Périmètre d'analyse impossible à délimiter → demande de clarification architecturale avant de continuer.
- Modèle de menaces obsolète (> 6 mois) détecté → déclencher une revue complète avant toute approbation de déploiement majeur.
