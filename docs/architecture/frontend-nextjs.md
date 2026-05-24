# Frontend — Next.js

## Stack

- **Framework** : Next.js (dernière version stable)
- **Router** : App Router — exclusivement. Le Pages Router n'est pas utilisé.
- **Langage** : TypeScript strict
- **Localisation dans le monorepo** : `apps/web/`

## Multi-tenant et routing par domaine

La plateforme opère sous plusieurs domaines (`efatourati.ma`, `efatourati.com`, etc.). Le routing multi-tenant peut être géré par :

- **Domaine principal unique** avec sélection de tenant dans l'application
- **Sous-domaines par tenant** (ex. `{tenant}.efatourati.ma`) — `[assumption : approche à décider]`

La stratégie de routing tenant est une décision d'architecture non finalisée. Ne pas implémenter une approche sans validation explicite.

## Architecture des composants

### Défaut : Server Components

Tous les composants sont des Server Components par défaut. Un composant devient Client Component uniquement via `'use client'` avec une justification explicite.

| Type | Quand l'utiliser |
|---|---|
| Server Component | Récupération de données, rendu statique, logique sans interactivité |
| Client Component | État local, event handlers, hooks browser (`useEffect`, `useState`) |

### Contexte tenant dans les layouts

Le contexte tenant (tenant ID, profil, année fiscale active) est résolu par le middleware Next.js et injecté dans les layouts serveur. Les composants enfants le reçoivent par props ou via un contexte React côté client.

```
middleware.ts  →  résout tenantId depuis session/cookie
  └─ layout.tsx (Server Component)  →  charge les données tenant
       └─ TenantProvider (Client Component)  →  expose le contexte React
            └─ composants enfants
```

## Internationalisation

La plateforme supporte **l'arabe** et le **français**. `[assumption : langues supplémentaires à valider]`

- L'arabe est une langue RTL — l'interface doit supporter les deux directions (LTR/RTL).
- La langue est persistée dans la préférence utilisateur, pas uniquement dans l'URL.
- Les chaînes de traduction ne sont pas hardcodées dans les composants. Un système i18n dédié est utilisé.

## Authentification

> `[assumption]` La stratégie d'authentification exacte est à définir. Contraintes connues :

- L'authentification est gérée côté serveur (pas de token exposé côté client).
- La session inclut : userId, tenantId actif, rôle dans le tenant actif, fiscalYearId actif.
- Le changement de tenant actif régénère la session.
- Le middleware Next.js protège toutes les routes authentifiées et redirige vers la page de connexion si la session est absente ou invalide.

## Récupération de données

- Les Server Components récupèrent les données directement via les Server Actions ou en appelant l'API NestJS côté serveur.
- Les Client Components utilisent des hooks dédiés qui appellent des Route Handlers ou des Server Actions.
- Pas d'appel direct depuis un Client Component vers l'API NestJS externe. `[à confirmer selon l'approche retenue]`

## Performance et sécurité

- Les headers de sécurité (CSP, HSTS, X-Frame-Options) sont configurés dans `next.config.js`.
- Les routes API Next.js (Route Handlers) valident systématiquement l'authentification et le contexte tenant.
- Les données sensibles (NIF, ICE, données financières) ne sont jamais exposées dans les URLs ou les query params.
