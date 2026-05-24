# Control Evidence

Ce dossier centralise les preuves de contrôle collectées pour démontrer la conformité réglementaire de la plateforme de facturation électronique marocaine lors des inspections DGI, CNDP, et des audits ISO 27001.

## Principe

Une preuve de contrôle est un artefact horodaté, signé, et vérifiable qui démontre qu'un contrôle de sécurité ou de conformité est en place et opérationnel. Les preuves sont la matière première de tout audit réglementaire.

## Catégories de preuves

### Preuves de conformité DGI
- Rapports de validation de format de facture
- Journaux de transmission réussie vers la DGI
- Accusés de réception DGI archivés
- Rapports de test des mécanismes de signature

### Preuves de contrôle DGSSI / ISO 27001
- Rapports de revues de sécurité du code (skill `cryptographic-review`)
- Résultats de scans de vulnérabilités
- Journaux de rotation des clés et des certificats
- Rapports d'audit d'accès aux systèmes sensibles
- Preuves de formation à la sécurité de l'équipe

### Preuves de conformité CNDP
- Registre des traitements (version datée)
- PIA (Privacy Impact Assessments) complétées
- Déclarations CNDP et accusés de réception
- Journaux de traitement des demandes de droits des personnes

### Preuves de release sécurisée
- Rapports de `secure-release` par version déployée
- Résultats de `compliance-review` pré-déploiement
- Approbations d'agents documentées

## Structure du dossier

```
control-evidence/
├── README.md                    # Ce fichier
├── dgi/                         # Preuves DGI par période
├── dgssi/                       # Preuves DGSSI / sécurité technique
├── iso27001/                    # Preuves par contrôle de l'Annexe A
│   ├── org/                     # Contrôles organisationnels
│   ├── people/                  # Contrôles personnes
│   ├── physical/                # Contrôles physiques
│   └── tech/                    # Contrôles technologiques
├── cndp/                        # Preuves de conformité CNDP
└── releases/                    # Rapports de release sécurisée par version
```

## Conventions de nommage

```
YYYY-MM-DD_type-controle_description-courte.md
```

Exemples :
- `2026-05-24_iso27001-A.8.24_algorithmes-approuves.md`
- `2026-05-24_dgi_validation-format-facture-v2.md`
- `2026-05-24_cndp_pia-module-archivage.md`

## Conservation

Les preuves de contrôle sont conservées selon les durées légales applicables :
- Preuves fiscales DGI : 10 ans minimum
- Preuves ISO 27001 : durée du cycle de certification + 3 ans
- Preuves CNDP : durée du traitement + 5 ans

Aucune preuve ne peut être supprimée sans approbation explicite de l'agent `audit` et documentation de la procédure de purge.
