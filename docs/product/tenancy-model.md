# Tenancy Model

## Définition d'un tenant

Un **tenant** est l'unité d'isolation principale de la plateforme. Il représente une entité légale ou un professionnel indépendant. Toutes les données métier, tous les documents fiscaux, et tous les accès utilisateurs sont scopés à un tenant.

## Profils de tenant

Le profil d'un tenant est défini à la création et **ne peut pas être modifié après validation** `[assumption : à confirmer selon les règles métier]`.

| Profil | Représente | Capacités spécifiques |
|---|---|---|
| `fiduciaire` | Cabinet comptable, expert-comptable | Peut initier/accepter une FiduciaireRelationship avec des tenants `company` |
| `auto-entrepreneur` | Indépendant sous régime simplifié | Obligations comptables allégées — *périmètre fonctionnel précis à définir* |
| `company` | PME, grande entreprise | Comptabilité complète, peut inviter une fiduciaire ou accepter d'en être géré |

## Membership et multi-appartenance

- Un **utilisateur peut appartenir à plusieurs tenants** avec des rôles différents dans chacun.
- La session utilisateur doit toujours avoir un **tenant actif sélectionné**. Le changement de contexte tenant est une action explicite de l'utilisateur.
- Les données et actions d'un tenant ne sont jamais visibles depuis un autre tenant, sauf via FiduciaireRelationship.

## Premier utilisateur — devenir admin

- Le **premier utilisateur** qui rejoint un tenant nouvellement créé obtient automatiquement le statut d'administrateur de ce tenant.
- Ce mécanisme s'applique à la création du tenant (l'utilisateur qui crée le tenant devient admin).
- La promotion d'autres utilisateurs comme admin est une action réservée aux admins existants.

## Isolation des données

L'isolation est une contrainte de sécurité et de conformité, pas une simple règle applicative.

- Le `tenantId` est obligatoire dans toutes les requêtes authentifiées.
- Les requêtes de base de données sont systématiquement filtrées par `tenantId`.
- La `FiduciaireRelationship` est le seul mécanisme légal de partage de données inter-tenant. Elle est explicite, acceptée, et révocable.

## Dualité fournisseur-client

Un tenant n'est pas exclusivement fournisseur ou exclusivement client — les deux rôles coexistent dans la même entité selon les transactions.

- Un tenant peut **émettre des factures** (rôle fournisseur) vers d'autres tenants inscrits sur la plateforme ou vers des tiers externes non inscrits.
- Un tenant peut **recevoir des factures** (rôle client) de fournisseurs enregistrés sur la plateforme.
- Les vues "Factures émises" et "Factures reçues" sont distinctes dans l'interface et dans les APIs. Elles ne se mélangent pas.

Cette dualité est valable pour tous les profils :

| Profil | Rôle fournisseur | Rôle client |
|---|---|---|
| `fiduciaire` | Facture ses honoraires aux clients entreprises | Reçoit des factures de ses propres prestataires |
| `company` | Facture ses clients (B2B/B2C) | Reçoit des factures de ses fournisseurs |
| `auto-entrepreneur` | Facture ses clients | Reçoit des factures de ses prestataires |

Le contexte d'année fiscale s'applique différemment selon le rôle :
- Les factures **émises** sont scopées à l'année fiscale active du **fournisseur** au moment de l'émission.
- Les factures **reçues** sont consultables dans la vue client indépendamment du contexte d'année fiscale actif du client.

## Provisionnement d'un tenant

> `[assumption]` Le flux exact de création de tenant (inscription, vérification, activation) est à définir. Les éléments suivants sont des hypothèses de travail :

1. Un utilisateur s'inscrit sur la plateforme.
2. Il crée un tenant en choisissant un profil et en renseignant les informations légales (NIF, ICE selon profil).
3. Une vérification (manuelle ou automatique) peut être requise selon le profil.
4. À l'activation, l'utilisateur devient tenant admin.

## Cycle de vie d'un tenant

| Statut | Description |
|---|---|
| `active` | Opérationnel, accès complet |
| `suspended` | Accès bloqué temporairement (défaut de paiement, incident) |
| `terminated` | Compte fermé, données en rétention légale |

> La politique de rétention des données après `terminated` est soumise aux durées légales marocaines. **Requires official DGI/CNDP verification.**
