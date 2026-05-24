# DGI — Facturation Électronique

> **Requires official DGI verification** — Ce document synthétise les exigences connues de la DGI pour la facturation électronique au Maroc. Il ne se substitue pas aux textes officiels. Toute exigence spécifique (format, délais, endpoints) doit être vérifiée contre la documentation officielle DGI avant implémentation.

## Contexte

La Direction Générale des Impôts impose des règles strictes pour la génération, la signature, la transmission, et l'archivage des factures électroniques dans le cadre de la réforme de dématérialisation fiscale marocaine.

## Exigences confirmées (principes généraux)

Ces principes sont généralement établis dans les régimes de facturation électronique. Leur application exacte au cadre marocain **requiert vérification officielle**.

### Format des factures

- Les factures électroniques respectent un format structuré défini par la DGI.
- Les champs obligatoires incluent : identifiants fiscaux des parties (NIF, ICE), numérotation séquentielle, dates, montants HT/TTC, taux et montant de TVA.
- Le format exact (XML, JSON, autre) **requires official DGI verification**.

### Signature électronique

- Les factures électroniques sont signées avec un certificat qualifié émis par une autorité de certification reconnue.
- La signature garantit l'intégrité et la non-répudiation du document.
- Les spécifications techniques de la signature (algorithme, format de signature, horodatage) **requires official DGI verification**.

### Transmission

- Les factures transmises à la DGI le sont via un canal sécurisé défini par la DGI.
- Les accusés de réception DGI sont conservés comme preuves de transmission.
- Les délais de transmission et les procédures de correction en cas de rejet **requires official DGI verification**.

### Archivage

- Les factures électroniques sont archivées pour une durée minimale définie par la loi fiscale marocaine.
- La durée généralement citée est de **10 ans** pour les documents fiscaux — **requires official DGI verification**.
- L'intégrité des archives doit être garantie sur toute la durée de rétention.
- Les archives doivent être accessibles en cas d'inspection fiscale.

## Profils d'assujettis

> **Requires official DGI verification** — Les obligations varient selon la taille et le type de contribuable.

| Profil | Obligations attendues |
|---|---|
| Grande entreprise | Facturation électronique obligatoire (calendrier à vérifier) |
| PME | Obligation progressive (calendrier à vérifier) |
| Auto-entrepreneur | Régime simplifié — obligations spécifiques à vérifier |

## Implication pour le modèle multi-tenant

- Chaque tenant (fiduciaire, auto-entrepreneur, company) a des obligations DGI différentes selon son profil.
- La plateforme doit adapter les workflows de facturation au profil du tenant.
- Un fiduciaire agissant pour le compte d'une entreprise cliente soumet les factures au nom de l'entreprise, pas en son propre nom — les identifiants fiscaux utilisés sont ceux du tenant company.

## Clearance bidirectionnelle — implications DGI

> **Requires official DGI verification** — Le modèle exact de clearance bidirectionnelle côté DGI, et notamment les obligations du destinataire, doit être vérifié contre les textes officiels en vigueur.

La nature bidirectionnelle de la plateforme a des implications directes sur la conformité DGI :

- **Identités dans la facture** : La facture clearée doit identifier à la fois le fournisseur (émetteur) et le client (destinataire) via leurs identifiants fiscaux respectifs (NIF, ICE). Les deux identités sont soumises à validation lors de la clearance.
- **Transmission au destinataire** : Après clearance, la facture doit être transmise au client. Les modalités exactes (push DGI, pull via portail, transmission plateforme) — **requires official DGI verification**.
- **Obligations du client** : Si et comment le client destinataire doit formellement répondre à la DGI (accusé de réception, validation, contestation) — **requires official DGI verification**.
- **Correction post-clearance** : Les procédures DGI pour corriger une facture déjà clearée (avoir, annulation, nouvelle soumission) — **requires official DGI verification**.
- **Client externe hors plateforme** : Les obligations qui incombent au fournisseur lorsque le client destinataire n'est pas inscrit sur la plateforme — **requires official DGI verification**.

## Lien avec le suivi opérationnel

Les exigences DGI détaillées, les contrôles en place, et les preuves de conformité sont tracés dans :
- `compliance/dgi/` — cartographie opérationnelle des exigences
- `docs/control-evidence/dgi/` — preuves de conformité collectées
