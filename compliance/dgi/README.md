# Conformité DGI — Direction Générale des Impôts

## Contexte réglementaire

La Direction Générale des Impôts du Maroc impose des exigences strictes pour la facturation électronique dans le cadre de la réforme de la TVA et de la dématérialisation fiscale. Ce dossier trace la conformité de la plateforme à ces exigences.

## Périmètre de conformité

### Format des factures électroniques
- Structure et champs obligatoires définis par la DGI
- Identifiants fiscaux (NIF, ICE) : format et validation
- Numérotation séquentielle et non altérable des factures
- Mentions légales obligatoires

### Signature électronique
- Certificat qualifié émis par une autorité de certification reconnue DGI
- Algorithmes de signature conformes aux spécifications DGI
- Horodatage qualifié des signatures
- Non-répudiation garantie

### Transmission vers la DGI
- Protocoles de communication définis par la DGI
- Délais de transmission réglementaires
- Accusés de réception et gestion des rejets
- Procédures de correction et de réémission

### Archivage fiscal
- Durées de conservation légales (actuellement 10 ans)
- Intégrité et lisibilité garanties sur toute la durée de rétention
- Accessibilité en cas d'inspection fiscale

## Structure du dossier

```
dgi/
├── README.md                    # Ce fichier
├── requirements/                # Exigences DGI détaillées et statut
├── format-specs/                # Spécifications de format de facture
└── integration/                 # Documentation d'intégration API DGI
```

## Statut de conformité

> Les documents de cartographie détaillée sont créés au fur et à mesure de l'implémentation et des validations réglementaires.
