# Conformité ISO/IEC 27001:2022

## Contexte

La certification ISO 27001 est l'objectif de gouvernance de sécurité de la plateforme. Elle structure le Système de Management de la Sécurité de l'Information (SMSI) et fournit le cadre de référence pour les contrôles de sécurité (Annexe A, 93 contrôles organisés en 4 thèmes).

## Périmètre du SMSI

Le périmètre couvre : la plateforme de facturation électronique, ses composants d'infrastructure, ses interfaces avec la DGI, et les processus de traitement des données fiscales.

## Thèmes de l'Annexe A (ISO 27001:2022)

### 5 — Contrôles organisationnels (37 contrôles)
Politiques de sécurité, organisation, gestion des actifs, contrôle d'accès, cryptographie, sécurité des fournisseurs, gestion des incidents, continuité d'activité, conformité.

### 6 — Contrôles liés aux personnes (8 contrôles)
Sélection, conditions d'emploi, sensibilisation, formation, processus disciplinaire, après la fin d'emploi, télétravail.

### 7 — Contrôles physiques (14 contrôles)
Périmètre de sécurité physique, contrôle d'accès physique, sécurité des bureaux, protection contre les menaces environnementales, travail dans les zones sécurisées, bureau propre.

### 8 — Contrôles technologiques (34 contrôles)
Appareils d'utilisateur final, droits d'accès privilégiés, gestion des clés cryptographiques, protection contre les malwares, journalisation, monitoring, gestion des vulnérabilités, développement sécurisé, tests de sécurité.

## Structure du dossier

```
iso27001/
├── README.md                    # Ce fichier
├── scope.md                     # Périmètre du SMSI
├── risk-register.md             # Registre des risques
├── annex-a/                     # Cartographie des 93 contrôles
│   ├── org-controls.md          # Thème 5 — Organisationnels
│   ├── people-controls.md       # Thème 6 — Personnes
│   ├── physical-controls.md     # Thème 7 — Physiques
│   └── tech-controls.md         # Thème 8 — Technologiques
└── soa.md                       # Statement of Applicability
```

## Statut

> La cartographie des contrôles de l'Annexe A est construite en parallèle de l'implémentation. Les preuves de contrôle sont collectées dans `../../docs/control-evidence/`.
