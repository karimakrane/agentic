# Clearance Flow — Fournisseur ↔ Client

## Principe

e-fatourati est une plateforme **bidirectionnelle** : un tenant peut émettre des factures (fournisseur) et en recevoir (client). La clearance DGI est l'étape centrale commune aux deux flux. Un même tenant est fournisseur dans certaines transactions et client dans d'autres.

La clearance DGI valide, horodate, et signe la facture avant qu'elle n'atteigne le destinataire. Aucune facture ne peut être présentée à un client sans avoir été clearée.

> **Requires official DGI verification** — Les modalités exactes de transmission après clearance (push DGI, pull plateforme, portail), les obligations de réponse client, et les procédures de correction sont à vérifier contre les textes officiels DGI.

---

## Flux outbound — Le tenant est fournisseur

```
[Tenant Fournisseur]
  1. Crée la facture (draft)
  2. Valide le format DGI (localement)
  3. Signe électroniquement (certificat qualifié)
  4. Soumet à la DGI pour clearance
           │
           ├── [DGI accepte] ─────────────────────────────────────┐
           │     Référence de clearance attribuée                  │
           │                                                       ▼
           │                                          Facture clearée transmise au client
           │                                                       │
           │                                        ┌──────────────┼─────────────────┐
           │                                   [Accepte]      [Dispute]          [Rejette]
           │                                        │              │                   │
           │                                 Facture acceptée  Litige ouvert    Rejet notifié
           │                                                    fournisseur       fournisseur
           │                                                    notifié
           │
           └── [DGI rejette]
                 Erreur de clearance notifiée au fournisseur
                 → Corriger et resoumettre
```

---

## Flux inbound — Le tenant est client

```
[Facture clearée par DGI]
  → Transmise au Tenant Client
  → Notification (in-app + email)
  → Le client consulte la facture dans sa vue "Factures reçues"
           │
           ├── [Accepte]
           │     InvoiceResponse(accepted)
           │     Flux de paiement activé si applicable
           │
           ├── [Dispute]
           │     InvoiceResponse(disputed) + motif obligatoire
           │     Litige ouvert — fournisseur notifié
           │     Procédure de résolution engagée
           │
           └── [Rejette]
                 InvoiceResponse(rejected) + motif obligatoire
                 Fournisseur notifié
                 Fournisseur peut émettre un avoir + nouvelle facture corrigée
```

---

## Statuts du cycle de vie d'une facture

Une facture porte deux axes de statut indépendants.

### Axe clearance DGI (`clearanceStatus`)

| Statut | Description |
|---|---|
| `draft` | Créée, non soumise |
| `pending_signature` | En attente de signature électronique |
| `pending_clearance` | Soumise à la DGI, réponse en attente |
| `cleared` | Clearance accordée, référence DGI attribuée |
| `clearance_rejected` | Rejetée par la DGI — correction requise avant resoumission |

### Axe réponse client (`deliveryStatus`)

| Statut | Description | Prérequis |
|---|---|---|
| `pending_delivery` | Clearée, en cours de transmission | `clearanceStatus = cleared` |
| `delivered` | Transmise au client | — |
| `acknowledged` | Client a accusé réception | — |
| `accepted` | Client a accepté formellement | — |
| `disputed` | Litige ouvert par le client | — |
| `rejected` | Rejetée par le client | — |

La progression du `deliveryStatus` est pilotée par les `ClearanceEvent` correspondants. Aucune mise à jour directe n'est possible depuis l'extérieur du système.

---

## Clients externes (hors plateforme)

Un fournisseur peut émettre une facture à destination d'un client qui n'est **pas inscrit** sur e-fatourati.

- La facture est clearée par la DGI normalement.
- La facture clearée est transmise au client par un canal externe (email avec PDF signé, portail invité, autre).
- La réponse client (acceptation, dispute) est gérée hors plateforme dans ce cas, sauf si un portail invité est prévu.

> `[assumption]` Le mécanisme de transmission et de réponse pour les clients externes est à définir.

---

## Correction et resoumission

| Scénario | Action fournisseur |
|---|---|
| Rejet DGI — erreur de format | Corriger la facture, resoumettre |
| Rejet DGI — montants incorrects | Corriger les montants, resoumettre |
| Refus client — erreur de montant | Émettre un avoir + nouvelle facture corrigée |
| Refus client — prestation non reçue | Engager la procédure de litige |
| Dispute client non résolue | Procédure de médiation — `[assumption]` : à définir |

> **Requires official DGI verification** — Les procédures réglementaires exactes de correction et de resoumission après clearance (avoir, annulation, resoumission) doivent être vérifiées.

---

## Workflow litige

> `[assumption]` Le workflow de litige est à définir. Éléments attendus :

1. Client initie le litige avec motif documenté et horodatage.
2. Fournisseur reçoit une notification et peut répondre.
3. Resolution possible par : avoir + nouvelle facture, acceptation après clarification, médiation.
4. Traçabilité complète dans les `ClearanceEvent` de la facture.
5. Litige non résolu après délai → `[assumption]` escalade selon procédure à définir.

---

## Vue fournisseur vs vue client

L'interface expose deux vues distinctes pour le même tenant :

| Vue | Contenu | Filtres disponibles |
|---|---|---|
| **Factures émises** | Factures où `supplierId = tenantCourant` | Par `clearanceStatus`, par client, par période |
| **Factures reçues** | Factures où `clientId = tenantCourant` | Par `deliveryStatus`, par fournisseur, par période |

Ces deux vues sont indépendantes. Une même transaction n'apparaît jamais dans les deux vues du même tenant simultanément.
