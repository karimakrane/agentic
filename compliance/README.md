# Compliance

Ce dossier centralise la cartographie de conformité réglementaire de la plateforme de facturation électronique marocaine.

## Périmètre réglementaire

| Référentiel | Autorité | Domaine | Dossier |
|---|---|---|---|
| Facturation électronique | DGI — Direction Générale des Impôts | Format, signature, transmission, archivage fiscal | `dgi/` |
| Sécurité des SI | DGSSI — Direction Générale de la Sécurité des Systèmes d'Information | Contrôles de sécurité, algorithmes, certification | `dgssi/` |
| Protection des données | CNDP — Commission Nationale de contrôle de la Protection des Données | Traitements PII, droits des personnes, registre | `cndp/` |
| Management de la sécurité | ISO/IEC 27001:2022 | Système de management, Annexe A, risques | `iso27001/` |
| Données européennes | GDPR — Règlement (UE) 2016/679 | Clients européens, transferts hors UE | `gdpr/` |

## Utilisation

- Chaque sous-dossier contient la cartographie des exigences, le statut des contrôles, et les référentiels de conformité.
- Les preuves de contrôle collectées sont archivées dans `../docs/control-evidence/`.
- Les décisions architecturales à impact réglementaire sont documentées dans `../docs/decisions/`.
- Toute modification dans ce dossier requiert l'approbation de l'agent `compliance`.
