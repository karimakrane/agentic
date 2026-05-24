# e-fatourati — Product Overview

## Application

**e-fatourati** est une plateforme SaaS multi-tenant de facturation électronique et de clearance bidirectionnelle fournisseur-client dédiée au marché marocain.

**Domaines** : `efatourati.ma` · `efatourati.com` · `e-fatourati.ma` · `e-fatourati.com`

## Proposition de valeur

La plateforme est bidirectionnelle : un tenant peut à la fois **émettre** des factures (rôle fournisseur) et **en recevoir** (rôle client). Les deux flux passent par la clearance DGI.

- **Émettre des factures** : génération, signature électronique, soumission à la DGI pour clearance, transmission au client
- **Recevoir des factures** : réception des factures clearées par la DGI, accusé de réception, validation ou dispute
- **Clearance DGI bidirectionnelle** : chaque facture est clearée par la DGI avant d'atteindre le destinataire — dans les deux sens
- **Gestion comptable** : dans un contexte d'année fiscale obligatoire et strict
- **Gestion fiduciaire** : les fiduciaires gèrent les dossiers de leurs clients entreprises depuis une interface unifiée
- **Archivage légal** : conservation traçable des documents fiscaux sur les durées légales

## Profils de tenants

| Profil | Description | Rôles dans les transactions |
|---|---|---|
| **Fiduciaire** | Cabinet comptable ou expert-comptable | Fournisseur (honoraires) + client (achats) + gestionnaire de clients entreprises |
| **Auto-entrepreneur** | Indépendant sous régime simplifié | Fournisseur (prestations) + client (achats) |
| **Entreprise** | PME ou grande entreprise | Fournisseur (ventes) + client (achats) |

## Clearance bidirectionnelle

La clearance est le processus central de la plateforme. Il est identique quel que soit le sens de la transaction :

```
[Fournisseur] → émet → [Facture] → soumet → [DGI : clearance]
                                                    │
                                     ← facture clearée transmise ←
                                                    │
                                             [Client] → répond (accepte / dispute / rejette)
```

- Un tenant est **fournisseur** dans les factures qu'il émet.
- Le même tenant est **client** dans les factures qu'il reçoit.
- Un fournisseur peut émettre vers un **tenant inscrit** sur la plateforme ou vers un **tiers externe** (client non inscrit).

## Modèle multi-tenant

- Un utilisateur peut appartenir à plusieurs tenants simultanément.
- Chaque tenant a un profil unique (`fiduciaire`, `auto-entrepreneur`, ou `company`).
- Le premier utilisateur créant un tenant devient automatiquement administrateur de ce tenant.
- Les données de chaque tenant sont strictement isolées.
- Un fiduciaire peut accéder aux données d'une entreprise cliente uniquement via une `FiduciaireRelationship` explicitement acceptée.
- La dualité fournisseur-client est valable pour tous les profils — aucun profil n'est exclusivement l'un ou l'autre.

## Console d'administration interne

La plateforme dispose d'une console d'administration interne réservée aux opérateurs de e-fatourati. Elle est distincte de l'interface tenant et gère le cycle de vie de la plateforme elle-même.

## Contexte réglementaire

- Facturation électronique et clearance sous exigences DGI (Direction Générale des Impôts, Maroc)
- Sécurité des systèmes sous supervision DGSSI
- Protection des données personnelles sous loi 09-08 (CNDP)
- Objectif de certification ISO 27001

## Documents de référence

- [Domain Model](domain-model.md)
- [Clearance Flow](clearance-flow.md)
- [Tenancy Model](tenancy-model.md)
- [User Roles](user-roles.md)
- [Fiscal Year Context](fiscal-year-context.md)
- [Fiduciaire–Company Workflow](fiduciaire-company-workflow.md)
- [Admin Console](admin-console.md)
