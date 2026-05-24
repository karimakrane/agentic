# Contexte Réglementaire Marocain

> Ce document présente le cadre réglementaire marocain applicable à e-fatourati. Il est de nature documentaire et ne constitue pas un avis juridique. Les détails opérationnels sont tracés dans le dossier `compliance/`.

## Autorités de régulation concernées

### DGI — Direction Générale des Impôts

- **Rattachement** : Ministère de l'Économie et des Finances
- **Rôle** : Administration fiscale nationale
- **Pertinence pour e-fatourati** : Exigences de facturation électronique, TVA, archivage fiscal, formats de transmission

### DGSSI — Direction Générale de la Sécurité des Systèmes d'Information

- **Rattachement** : Administration de la Défense Nationale
- **Rôle** : Autorité nationale de cybersécurité
- **Pertinence pour e-fatourati** : Algorithmes cryptographiques approuvés, sécurité des systèmes d'information, notification d'incidents

### CNDP — Commission Nationale de contrôle de la Protection des Données à caractère personnel

- **Cadre légal** : Loi 09-08 relative à la protection des personnes physiques à l'égard du traitement des données à caractère personnel
- **Rôle** : Régulateur de la protection des données personnelles au Maroc
- **Pertinence pour e-fatourati** : Traitements de données personnelles (NIF, ICE, coordonnées), droits des personnes, registre des traitements

## Contexte de la réforme de la facturation électronique

> **Requires official verification** — La réforme marocaine de la facturation électronique est en cours. Les informations suivantes reflètent l'état connu à la date de rédaction et doivent être vérifiées contre les textes officiels en vigueur.

Le Maroc engage depuis plusieurs années une réforme de dématérialisation fiscale visant à :

- Réduire la fraude à la TVA
- Améliorer la traçabilité des transactions commerciales
- Moderniser les échanges entre contribuables et administration fiscale

Les obligations de facturation électronique s'appliquent progressivement selon la taille des entreprises. Les modalités exactes (calendrier, périmètre, formats) **doivent être vérifiées directement auprès de la DGI**.

## Cadre légal de référence

> Les références légales suivantes sont indicatives. Vérifier les versions consolidées en vigueur.

| Texte | Domaine |
|---|---|
| Code Général des Impôts (CGI) | TVA, IS, IR, obligations fiscales |
| Loi 09-08 | Protection des données personnelles |
| Dahirs et circulaires DGI | Modalités de facturation électronique |
| Référentiels DGSSI | Sécurité des systèmes d'information |

## Relation entre les référentiels

```
ISO 27001  ──── cadre de management de la sécurité
    │
    ├── DGSSI  ──── exigences cryptographiques et de sécurité nationales
    │
    └── DGI    ──── exigences fiscales et de transmission
         │
         └── CNDP / GDPR  ──── protection des données dans les documents fiscaux
```

## Documents de référence détaillés

- [DGI](dgi.md) — exigences de facturation électronique
- [DGSSI](dgssi.md) — contrôles de sécurité
- [CNDP](cndp.md) — protection des données
- [ISO 27001](iso27001.md) — gouvernance de la sécurité
- [GDPR](gdpr.md) — compatibilité pour les données européennes
