# Project Overview

Ce dépôt est le workspace de développement de **e-fatourati**, une plateforme SaaS multi-tenant de facturation électronique et de clearance bidirectionnelle fournisseur-client dédiée au marché marocain, opérée sous régime réglementé.

**Application** : e-fatourati
**Domaines** : `efatourati.ma` · `efatourati.com` · `e-fatourati.ma` · `e-fatourati.com`

La plateforme adresse trois profils de tenants :
- **Fiduciaires** — cabinets comptables gérant plusieurs entreprises clientes via relation explicite
- **Auto-entrepreneurs** — indépendants avec obligations fiscales simplifiées
- **Entreprises** — PME et grandes entreprises avec comptabilité et facturation complètes

Ce workspace couvre l'ensemble du cycle de vie de la conformité et du développement :

- **Compliance tracking** — cartographie des exigences DGI, DGSSI, CNDP, ISO 27001, GDPR
- **Architecture réglementée** — conception multi-tenant sous contraintes légales et sécuritaires
- **Implementation contrôlée** — développement avec traçabilité des décisions de conformité
- **Security controls** — modélisation des menaces, revue cryptographique, audit de sécurité
- **Privacy by design** — détection PII, isolation tenant, minimisation des données, droits des personnes
- **Audit & evidence** — collecte de preuves de contrôle, journaux d'audit immuables
- **Release workflows** — validation réglementaire pré-déploiement, rollback documenté

Toute modification du système doit être évaluée sous l'angle de la conformité réglementaire **avant** d'être évaluée sous l'angle technique. Les exigences légales ne sont pas négociables.

---

# Technical Stack

Baseline technique confirmée. Les décisions non finalisées sont explicitement marquées. Ne pas inventer ni assumer une technologie non listée ici.

## Confirmé

| Couche | Technologie | Précisions |
|---|---|---|
| Monorepo | Structure monorepo | Outillage (Nx / Turborepo / pnpm workspaces) non encore décidé |
| Frontend | Next.js dernière version stable | App Router — pas de Pages Router |
| Backend | NestJS | Architecture modulaire par domaine |
| Infrastructure locale | Docker | `docker-compose` pour le développement local |

## Décisions en attente

Les points suivants ne sont pas finalisés. Aucun code ni documentation ne doit les assumer avant décision explicite :

- Outil de gestion du monorepo (Nx, Turborepo, pnpm workspaces natif)
- Base de données (technologie, topology, ORM)
- Message broker / queue
- Infrastructure de déploiement production (cloud provider, orchestrateur)
- CDN et stratégie de hosting frontend
- Service de gestion des secrets et HSM en production
- Stratégie d'envoi d'emails transactionnels
- Observability stack (logs, métriques, traces)

---

# Agent Operating Rules

Ces règles s'appliquent à tout agent opérant dans ce dépôt. Elles sont non négociables et s'appliquent sans exception.

## Analyse de conformité avant toute action

- Avant toute modification, identifier les réglementations applicables : DGI, DGSSI, CNDP, ISO 27001, GDPR.
- Toute fonctionnalité touchant à la facturation, la signature, la transmission ou l'archivage doit être validée contre les spécifications DGI en vigueur.
- Tout traitement de données personnelles déclenche automatiquement une analyse CNDP/GDPR.

## Plan obligatoire avant changement structurel

- Pour tout changement affectant : schémas de données, flux de transmission, mécanismes de signature, interfaces avec la DGI — exposer un plan documenté avant exécution.
- Le plan doit inclure : périmètre réglementaire impacté, analyse de risque, stratégie de rollback, et références aux contrôles ISO 27001 concernés.
- Attendre validation explicite d'un agent `compliance` ou `audit` avant d'engager le changement.

## Principe de changement minimal et traçable

- Préférer la modification chirurgicale à la réécriture. Chaque ligne modifiée doit pouvoir être justifiée en audit.
- Tout changement est accompagné d'une référence à l'exigence qui le motive (ticket, ADR, contrôle réglementaire).
- Les modifications sans justification traçable sont refusées.

## Intégrité du système réglementé

- Ne jamais supprimer de données ou de logs sans procédure de purge documentée et conforme CNDP.
- Ne jamais modifier les mécanismes de signature électronique sans revue cryptographique préalable.
- Ne jamais désactiver un contrôle de sécurité, même temporairement, sans procédure d'exception formelle.
- La compatibilité avec les APIs DGI est une contrainte absolue. Toute rupture bloque le déploiement.

## Communication et escalade

- Toute ambiguïté sur une exigence réglementaire est signalée immédiatement et bloque l'implémentation.
- Les décisions d'architecture impactant la conformité sont documentées dans un ADR horodaté.
- Les écarts par rapport aux contrôles ISO 27001 sont enregistrés comme des risques formels.

---

# Coding Standards

## Lisibilité et auditabilité

- Le code doit être lisible par un auditeur externe sans contexte supplémentaire.
- Les chemins critiques (signature, transmission DGI, archivage) font l'objet de commentaires expliquant la contrainte réglementaire, pas le mécanisme technique.
- Toute déviation par rapport aux standards est documentée avec la justification et la référence réglementaire.

## Nommage

- Les noms reflètent le domaine métier réglementé : `signFacture`, `transmitToDGI`, `archiveInvoice`, `validateTVA`.
- Pas d'abréviations dans les identifiants touchant aux données fiscales ou personnelles.
- Les constantes réglementaires (taux TVA, codes DGI, délais légaux) sont nommées explicitement et centralisées.

## Fonctions

- Une fonction = une responsabilité. Les fonctions touchant à des données réglementées sont encore plus strictement délimitées.
- Taille cible ≤ 30 lignes. Les fonctions de traitement de données personnelles sont isolées et clairement identifiées.
- La validation des entrées est obligatoire aux frontières du système : réception de factures, appels API DGI, ingestion de données externes.

## Duplication et réutilisation

- Les règles de validation réglementaire (formats DGI, contraintes CNDP) ne sont jamais dupliquées. Une source de vérité unique.
- Les schémas de données fiscales sont définis une fois et réutilisés par référence.

## Commentaires

- Commenter uniquement ce qui ne se déduit pas du code : contrainte réglementaire, délai légal imposé, format DGI spécifique.
- Les TODO non ticketés sont interdits dans le code de production.
- Les références aux articles de loi ou aux spécifications DGI sont les bienvenus comme commentaires.

## Séparation des responsabilités

- La logique fiscale (calcul TVA, génération des identifiants DGI) est strictement séparée de la logique de présentation et de persistance.
- Le traitement des données personnelles est isolé dans des modules dédiés soumis aux contrôles CNDP.
- Les clés cryptographiques ne transitent jamais dans la couche applicative métier.

## Modularité

- Les modules sont organisés par domaine réglementaire : `invoicing/`, `signature/`, `dgi-gateway/`, `archival/`, `privacy/`.
- Les dépendances entre modules sont unidirectionnelles et documentées dans les ADR.
- Aucun module transversal ne peut importer depuis un module de domaine.

## Conventions Next.js

- Les Server Components sont le défaut. Les Client Components sont opt-in via `'use client'` et justifiés explicitement.
- Le routing est App Router exclusivement. Aucun usage du Pages Router.
- Le contexte tenant (tenant ID, profil, année fiscale active) est résolu via middleware Next.js et disponible dans les layouts serveur.
- Les appels API depuis le serveur utilisent les Server Actions ou les Route Handlers. Les appels depuis le client passent par des hooks dédiés.
- L'internationalisation (arabe / français) est gérée au niveau routing, pas au niveau composant.

## Conventions NestJS

- Chaque domaine métier est un module NestJS autonome : `InvoicingModule`, `TenancyModule`, `AuthModule`, `FiscalYearModule`, `FiduciaireModule`, `ComplianceModule`, `AdminModule`.
- Le contexte tenant est injecté via un middleware dédié et accessible via un décorateur `@CurrentTenant()`.
- Les guards sont appliqués dans cet ordre : `JwtAuthGuard` → `TenantGuard` → `RoleGuard` → `FiscalYearGuard`.
- Les DTOs valident toutes les entrées via `class-validator`. Pas de validation manuelle dans les services.
- Les repositories encapsulent tout accès aux données. Les services ne connaissent pas le moteur de persistance.

---

# Architecture Principles

## Layered Architecture sous contrainte réglementaire

- Couches : API Gateway → Application → Domaine fiscal → Infrastructure → Systèmes DGI externes.
- Les dépendances pointent vers l'intérieur. La couche domaine fiscal ne connaît aucune infrastructure concrète.
- La couche d'intégration DGI est encapsulée et versionnée indépendamment pour absorber les évolutions réglementaires.

## Separation of Concerns

- Génération de factures, signature électronique, transmission DGI et archivage sont des domaines distincts avec des équipes et des cycles de déploiement indépendants.
- Les préoccupations transversales (audit trail, chiffrement, logging conforme) sont gérées par des composants dédiés non modifiables par les équipes produit.

## Dependency Boundaries

- Toute dépendance sur les systèmes DGI est encapsulée derrière une interface versionnée.
- Les bibliothèques cryptographiques sont approuvées par la DGSSI et ne sont pas remplacées sans processus de validation formel.
- Les dépendances tierces sont auditées trimestriellement pour CVE et conformité de licence.

## Explicit Interfaces

- Toutes les APIs exposées sont documentées avec leurs contrats de données, leurs codes d'erreur, et les obligations de rétention associées.
- Les changements d'interface vers la DGI suivent un processus de validation réglementaire préalable.
- Les interfaces internes exposant des données personnelles sont marquées `@pii-boundary` et soumises à revue CNDP.

## Observability-First Mindset

- Chaque transaction fiscale génère un journal d'audit immuable horodaté avec signature.
- Les métriques de disponibilité et de latence de transmission vers la DGI sont monitorées en temps réel.
- Les erreurs de validation réglementaire sont loguées avec le contexte complet, sans données personnelles.
- Les journaux d'audit sont conservés selon les durées légales de rétention fiscale marocaine.

## Security by Default

- Chiffrement de bout en bout pour toutes les données fiscales en transit et au repos.
- Authentification forte (certificats qualifiés) pour toute interaction avec les systèmes DGI.
- Le principe de moindre privilège s'applique à chaque composant, service, et opérateur humain.
- Les secrets cryptographiques sont gérés par un HSM ou un service de gestion de clés dédié.

## Testability by Design

- Chaque règle de validation DGI est couverte par un test unitaire référençant la spécification source.
- Les intégrations DGI sont testables via des environnements de recette isolés fournis par la DGI.
- Les scénarios de non-conformité sont des cas de test de première classe.

## Multi-Tenant Isolation

- Chaque tenant est un espace de données strictement isolé. L'accès croisé est interdit sauf via une `FiduciaireRelationship` explicitement établie et acceptée par les deux parties.
- Le `tenantId` est une composante obligatoire de toute requête authentifiée. Une requête sans contexte tenant valide est rejetée au niveau guard, pas au niveau service.
- Les migrations de schéma et les jobs batch vérifient systématiquement que les opérations ne débordent pas du périmètre tenant.
- Le contexte admin interne est totalement séparé du contexte tenant : authentification distincte, surfaces d'API distinctes, journaux d'audit distincts.

## Fiscal Year as Mandatory Context

- L'année fiscale est un contexte obligatoire pour toutes les opérations comptables, de facturation, de reporting, et de conformité.
- Aucun document fiscal ne peut être créé ou modifié sans année fiscale active et explicitement sélectionnée.
- Une année fiscale clôturée est en lecture seule. Toute tentative d'écriture sur une période clôturée est rejetée au niveau service.
- Le `fiscalYearId` est propagé dans tous les contextes d'exécution concernés (requêtes HTTP, jobs, événements).

## Monorepo Boundaries

- Structure cible : `apps/web` (Next.js), `apps/api` (NestJS), `packages/` (types, schémas, utilitaires purs partagés).
- `apps/web` ne peut pas importer depuis `apps/api` directement. La communication passe exclusivement par les APIs HTTP exposées.
- Les packages partagés ne contiennent que du code sans effet de bord : types TypeScript, schémas Zod, constantes.
- Les règles ESLint, TypeScript et Prettier sont définies au niveau racine et s'appliquent uniformément.

---

# Testing Requirements

## Nouvelles features

- Toute nouvelle fonctionnalité affectant le flux de facturation est accompagnée de tests couvrant : cas nominal, cas limite, et cas de rejet DGI.
- Les règles de validation réglementaire sont testées exhaustivement, pas par échantillonnage.

## Bug fixes

- Tout correctif est accompagné d'un test de régression. Si le bug concerne un flux réglementé, le test référence l'exigence DGI concernée.
- Les bugs de sécurité déclenchent une analyse d'impact réglementaire avant correction.

## Qualité des tests

- Les tests sont déterministes et reproductibles dans tous les environnements.
- Les tests de conformité DGI sont exécutés contre des fixtures officielles ou des données de test certifiées.
- Les tests impliquant des données personnelles utilisent exclusivement des données synthétiques.
- Zéro tolérance pour les flaky tests sur les chemins critiques (signature, transmission, archivage).

## Conditions de merge

- Aucun merge sans suite de tests verte.
- Toute PR impactant la conformité requiert l'approbation d'un agent `compliance` en plus de la revue technique.
- La couverture des chemins critiques réglementés ne peut pas régresser.

## Validation locale

- Exécuter la suite de conformité locale avant chaque push sur les branches réglementées.
- Les tests d'intégration DGI sont exécutés en environnement de staging avant merge vers `main`.

---

# Security Rules

## Secrets et credentials

- Zéro secret dans le code source, les commits, les logs, ou les variables d'environnement non chiffrées.
- Les certificats qualifiés et les clés de signature sont exclusivement gérés par un HSM certifié DGSSI.
- Les fichiers `.env` locaux sont dans `.gitignore`. Toute détection de secret en CI bloque le pipeline.

## Validation des entrées

- Toute donnée reçue de l'extérieur (clients, partenaires, DGI) est validée contre un schéma strict avant traitement.
- Les données fiscales sont validées contre les spécifications DGI officielles à la réception.
- Les erreurs de validation ne révèlent pas la structure interne du système.

## Moindre privilège

- Les services applicatifs n'ont accès qu'aux ressources nécessaires à leur fonction déclarée.
- Les accès aux journaux d'audit et aux archives fiscales sont restreints et traçés.
- Les droits d'accès sont révisés à chaque changement de périmètre et documentés dans le registre ISO 27001.

## Dépendances

- Toute bibliothèque cryptographique est approuvée par la DGSSI avant usage.
- L'audit des dépendances est automatisé en CI. Les CVE critiques bloquent le déploiement.
- Les dépendances de traitement de données personnelles sont évaluées pour conformité CNDP/GDPR.

## Logs et données sensibles

- Les logs ne contiennent jamais : NIF, numéros de facture en clair, données bancaires, données personnelles.
- Les identifiants fiscaux sont pseudonymisés dans les logs non-audit.
- Les journaux d'audit sont distincts des logs applicatifs et soumis à des contrôles d'accès stricts.

## Defaults sécurisés

- TLS 1.3 minimum pour toutes les communications. Les versions inférieures sont bloquées au niveau infrastructure.
- Les algorithmes cryptographiques suivent les recommandations DGSSI (et par extension ANSSI) en vigueur.
- Le mode debug est désactivé en production. Son activation requiert une procédure d'exception tracée.

---

# Deployment Constraints

## Compatibilité backward et réglementaire

- Toute modification du format des factures électroniques est backward compatible ou accompagnée d'une procédure de migration DGI validée.
- Les changements d'API vers la DGI sont coordonnés avec les équipes réglementaires avant déploiement.
- La règle : déprécier et notifier, puis migrer, puis supprimer — avec délais réglementaires respectés.

## Migrations

- Les migrations de schéma affectant des données fiscales ou personnelles sont réversibles, versionnées, et approuvées par les agents `compliance` et `audit`.
- Toute migration irréversible de données réglementées requiert une approbation formelle documentée.
- Les migrations sont testées en staging sur un jeu de données représentatif avant production.

## Rollback

- Chaque déploiement a une procédure de rollback testée et documentée.
- Le rollback ne doit jamais produire de perte de données fiscales ou d'incohérence dans les journaux d'audit.
- En cas d'incident post-déploiement affectant la transmission DGI, le rollback est immédiat et la DGI est notifiée selon les procédures légales.

## Changements d'infrastructure

- Toute modification d'infrastructure affectant la disponibilité ou la sécurité du système est documentée en IaC et soumise à revue.
- Les changements de configuration réseau, de pare-feu, ou de contrôle d'accès sont traités comme des changements de code : PR, revue, approbation.
- La continuité de service vers les systèmes DGI est une contrainte de déploiement de niveau critique.
