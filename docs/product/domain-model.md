# Domain Model

Entités métier principales de la plateforme e-fatourati et leurs relations. Ce document reflète les exigences produit confirmées. Les éléments marqués `[assumption]` nécessitent validation.

## Entités principales

### Tenant

Espace de travail isolé représentant une entité légale ou un professionnel.

| Attribut | Description |
|---|---|
| `id` | Identifiant unique du tenant |
| `profile` | `fiduciaire` \| `auto-entrepreneur` \| `company` |
| `name` | Raison sociale ou nom commercial |
| `status` | `active` \| `suspended` \| `terminated` |
| `fiscalId` | NIF (Numéro d'Identification Fiscale) — *format DGI à vérifier officiellement* |
| `iceNumber` | ICE (Identifiant Commun de l'Entreprise) — *applicable pour company et fiduciaire* |

### User

Personne physique pouvant appartenir à un ou plusieurs tenants avec des rôles différents par tenant.

| Attribut | Description |
|---|---|
| `id` | Identifiant unique de l'utilisateur |
| `email` | Identifiant d'authentification principal |
| `status` | `active` \| `suspended` \| `pending` |

### TenantMembership

Relation many-to-many entre User et Tenant, portant le rôle de l'utilisateur dans ce tenant.

| Attribut | Description |
|---|---|
| `userId` | Référence vers User |
| `tenantId` | Référence vers Tenant |
| `role` | Rôle dans ce tenant (voir [user-roles.md](user-roles.md)) |
| `isAdmin` | Le premier utilisateur d'un tenant a ce flag à `true` |
| `status` | `active` \| `suspended` \| `revoked` |

### FiscalYear

Contexte obligatoire pour toutes les opérations financières et fiscales.

| Attribut | Description |
|---|---|
| `id` | Identifiant unique |
| `tenantId` | Appartient à un tenant |
| `year` | Année civile (ex. 2025) — *voir [fiscal-year-context.md](fiscal-year-context.md) pour les cas non-calendaires* |
| `startDate` | Date de début |
| `endDate` | Date de fin |
| `status` | `open` \| `closed` \| `locked` |

### FiduciaireRelationship

Relation formelle entre un tenant fiduciaire et un tenant entreprise. Requiert acceptation explicite.

| Attribut | Description |
|---|---|
| `id` | Identifiant unique |
| `fiduciaireId` | Tenant initiateur (fiduciaire) |
| `companyId` | Tenant cible (entreprise) |
| `status` | `pending` \| `active` \| `suspended` \| `terminated` |
| `initiatedBy` | `fiduciaire` \| `company` — *flux bidirectionnel supporté* |
| `grantedScopes` | Périmètre d'accès accordé à la fiduciaire — *`[assumption]` : structure à définir* |

### Invoice

Document fiscal bidirectionnel. Porte à la fois la perspective fournisseur (émission, clearance DGI) et la perspective client (réception, réponse). Scopé à l'année fiscale du fournisseur émetteur.

| Attribut | Description |
|---|---|
| `id` | Identifiant interne |
| `supplierId` | Tenant fournisseur — émetteur de la facture |
| `clientId` | Tenant client destinataire — `null` si client externe hors plateforme |
| `externalClientRef` | Identité du client externe si `clientId` est `null` (raison sociale, NIF) |
| `fiscalYearId` | Année fiscale du fournisseur — obligatoire |
| `dgiClearanceRef` | Référence de clearance attribuée par la DGI — *format à vérifier officiellement* |
| `clearanceStatus` | `draft` \| `pending_signature` \| `pending_clearance` \| `cleared` \| `clearance_rejected` |
| `deliveryStatus` | `pending_delivery` \| `delivered` \| `acknowledged` \| `accepted` \| `disputed` \| `rejected` |
| `issuedAt` | Date d'émission |

### ClearanceEvent

Journal immuable des événements du cycle de clearance d'une facture. Un événement ne peut pas être modifié après création.

| Attribut | Description |
|---|---|
| `id` | Identifiant unique |
| `invoiceId` | Facture concernée |
| `eventType` | `submitted` \| `cleared` \| `clearance_rejected` \| `delivered` \| `response_received` |
| `occurredAt` | Horodatage de l'événement |
| `dgiPayload` | Réponse brute DGI — *`[assumption]` : format à vérifier officiellement* |
| `triggeredBy` | Référence système ou utilisateur ayant déclenché l'événement |

### InvoiceResponse

Réponse formelle du tenant client à une facture reçue et clearée par la DGI.

| Attribut | Description |
|---|---|
| `id` | Identifiant unique |
| `invoiceId` | Facture concernée |
| `clientTenantId` | Tenant client répondant — doit correspondre à `Invoice.clientId` |
| `responseType` | `accepted` \| `disputed` \| `rejected` |
| `reason` | Motif — obligatoire pour `disputed` et `rejected` |
| `respondedAt` | Horodatage de la réponse |

### ExternalParty

Tiers (client ou fournisseur) non inscrit sur la plateforme. Géré dans le répertoire privé du tenant qui l'a créé.

| Attribut | Description |
|---|---|
| `id` | Identifiant interne |
| `ownerTenantId` | Tenant propriétaire de cette entrée |
| `name` | Raison sociale |
| `fiscalId` | NIF ou équivalent — *format selon le pays, à vérifier officiellement* |
| `email` | Pour la transmission des factures clearées par email |
| `type` | `client` \| `supplier` \| `both` |

### InternalAdmin

Utilisateur de la console d'administration interne e-fatourati. Entité séparée du domaine tenant.

| Attribut | Description |
|---|---|
| `id` | Identifiant unique |
| `email` | Email opérateur |
| `role` | Rôle dans la console admin |

## Diagramme de relations

```
User ──< TenantMembership >── Tenant
                                │
          ┌─────────────────────┼──────────────────────┐
          │                     │                      │
    FiscalYear        FiduciaireRelationship      ExternalParty
          │              (fiduciaire ↔ company)
          │
    Invoice (supplierId / clientId)
          │
 ┌────────┴─────────┐
 │                  │
ClearanceEvent  InvoiceResponse
(journal DGI)   (réponse client)
```

## Contraintes métier confirmées

- Un User peut avoir zéro à N TenantMemberships actives simultanément.
- Un Tenant a exactement un profil (`fiduciaire` | `auto-entrepreneur` | `company`).
- Le premier User à rejoindre un Tenant devient automatiquement son admin (`isAdmin: true`).
- Une Invoice ne peut pas exister sans `fiscalYearId` valide et sans `supplierId`.
- Une Invoice a soit un `clientId` (tenant inscrit sur la plateforme) soit des données `externalClientRef` (client externe) — jamais les deux, jamais aucun des deux.
- Un `ClearanceEvent` est immuable après création — aucune modification n'est possible après insertion.
- Un `InvoiceResponse` ne peut être soumis que par le tenant identifié par `clientId` sur la facture concernée.
- La progression du `deliveryStatus` d'une Invoice est pilotée par les `ClearanceEvent` correspondants — pas de mise à jour directe depuis l'extérieur du système.
- Un `ExternalParty` appartient à un seul tenant (`ownerTenantId`) et n'est pas visible des autres tenants.
- Une FiduciaireRelationship ne peut exister qu'entre un tenant `fiduciaire` et un tenant `company`, **les deux devant être inscrits et actifs sur e-fatourati**. Un `ExternalParty` ne peut jamais être partie dans une FiduciaireRelationship — ni comme fiduciaire, ni comme company gérée.
- Une FiscalYear à statut `locked` ou `closed` rend ses documents associés non modifiables.
