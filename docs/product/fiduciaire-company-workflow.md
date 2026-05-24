# Fiduciaire–Company Workflow

## Concept

Une **FiduciaireRelationship** est une relation formelle, bidirectionnelle dans son initiation, et dont l'activation requiert l'acceptation explicite des deux parties. Elle permet à un cabinet fiduciaire d'accéder aux données comptables et fiscales d'une entreprise cliente via la plateforme.

## Règle fondamentale — périmètre e-fatourati

> **Les deux parties d'une FiduciaireRelationship doivent être des tenants inscrits et actifs sur e-fatourati.**

- Un fiduciaire **ne peut pas** gérer un compte externe non inscrit sur e-fatourati.
- Une entreprise **ne peut pas** être gérée par une fiduciaire externe à e-fatourati.
- Un `ExternalParty` (tiers non inscrit) ne peut jamais être partie dans une FiduciaireRelationship — ni comme fiduciaire, ni comme company gérée.
- L'`ExternalParty` est réservé exclusivement aux flux de facturation (émettre/recevoir des factures vers des tiers non inscrits). Il n'ouvre aucun droit de gestion de compte.

## Flux d'invitation — initiée par la fiduciaire

```
[Tenant Fiduciaire] ──invite──> [Invitation en attente] ──notifie──> [Tenant Company]
                                                                            │
                                                        ┌───────────────────┤
                                                   [Accepte]           [Refuse]
                                                        │                   │
                                              [Relation active]    [Invitation terminée]
```

### Étapes

1. L'admin du tenant fiduciaire saisit l'identifiant ou l'email du tenant company cible.
2. Une invitation est créée avec statut `pending`.
3. Le tenant company reçoit une notification (email + in-app).
4. L'admin du tenant company examine et accepte ou refuse.
5. En cas d'acceptation : la `FiduciaireRelationship` passe à `active`.
6. En cas de refus : l'invitation est terminée, aucune relation créée.

## Flux d'invitation — initiée par l'entreprise

```
[Tenant Company] ──invite──> [Invitation en attente] ──notifie──> [Tenant Fiduciaire]
                                                                            │
                                                        ┌───────────────────┤
                                                   [Accepte]           [Refuse]
                                                        │                   │
                                              [Relation active]    [Invitation terminée]
```

> Le flux est symétrique. La plateforme supporte les deux sens d'initiation. La règle d'acceptation s'applique dans les deux cas.

## Cycle de vie de la relation

| Statut | Description | Transition possible vers |
|---|---|---|
| `pending` | Invitation envoyée, en attente de réponse | `active`, `rejected`, `expired` |
| `active` | Relation établie, accès accordé | `suspended`, `terminated` |
| `suspended` | Accès temporairement bloqué | `active`, `terminated` |
| `terminated` | Relation définitivement close | — |

- La suspension ou la résiliation peut être initiée par l'une ou l'autre des parties.
- La résiliation révoque immédiatement tous les accès de la fiduciaire aux données de l'entreprise.

## Périmètre d'accès (grantedScopes)

> `[assumption]` La structure exacte des scopes est à définir. Les éléments suivants sont des hypothèses de travail.

Lors de l'acceptation d'une invitation, l'entreprise définit le périmètre d'accès accordé à la fiduciaire :

| Scope | Description |
|---|---|
| `invoices:read` | Lecture des factures |
| `invoices:write` | Création et modification de factures |
| `accounting:read` | Lecture des données comptables |
| `accounting:write` | Saisie comptable |
| `reports:read` | Accès aux rapports et états financiers |
| `fiscal:submit` | Soumission de déclarations fiscales |

Les scopes sont accordés au moment de l'acceptation et peuvent être modifiés ultérieurement par l'admin du tenant company.

## Contraintes métier

- Une invitation ne peut pas être envoyée deux fois au même tenant cible si une relation `pending` ou `active` existe déjà.
- Un tenant `fiduciaire` peut avoir des relations actives avec N tenants `company`.
- Un tenant `company` peut avoir des relations actives avec N tenants `fiduciaire` — `[assumption : à valider si un seul fiduciaire est autorisé]`.
- La fiduciaire ne peut agir pour le compte d'une entreprise que dans le périmètre des scopes accordés.
- Toutes les actions effectuées par la fiduciaire pour le compte d'une entreprise sont tracées avec l'identité de l'utilisateur fiduciaire (audit trail).

## Notifications

> `[assumption]` Les événements suivants déclenchent une notification. Les canaux (email, in-app, SMS) sont à définir.

- Invitation reçue (fiduciaire → company ou company → fiduciaire)
- Invitation acceptée
- Invitation refusée
- Relation suspendue
- Relation résiliée
