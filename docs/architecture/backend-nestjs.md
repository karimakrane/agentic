# Backend — NestJS

## Stack

- **Framework** : NestJS
- **Langage** : TypeScript strict
- **Localisation dans le monorepo** : `apps/api/`
- **Base de données** : non encore décidée — ne pas assumer de technologie

## Structure modulaire

L'API est organisée en modules NestJS par domaine métier. Chaque module est autonome et encapsule ses propres controllers, services, et repositories.

```
apps/api/src/
├── main.ts
├── app.module.ts              # Module racine
│
├── modules/
│   ├── auth/                  # Authentification, JWT, sessions
│   ├── tenancy/               # Gestion des tenants et memberships
│   ├── users/                 # Profils utilisateurs
│   ├── fiscal-year/           # Années fiscales
│   ├── invoicing/             # Génération et gestion des factures
│   ├── fiduciaire/            # Relations fiduciaire–company
│   ├── compliance/            # Contrôles de conformité DGI
│   └── admin/                 # Console d'administration interne
│
├── common/
│   ├── guards/                # AuthGuard, TenantGuard, RoleGuard, FiscalYearGuard
│   ├── decorators/            # @CurrentTenant(), @CurrentUser(), @Roles()
│   ├── middleware/            # Injection du contexte tenant
│   ├── interceptors/          # Logging, audit trail, transformation de réponse
│   └── filters/               # Gestion globale des exceptions
│
└── infrastructure/
    ├── database/              # Connexion et configuration DB (technologie TBD)
    ├── dgi-gateway/           # Client d'intégration DGI
    └── storage/               # Stockage des documents fiscaux
```

## Pipeline de requête et guards

Toutes les requêtes authentifiées traversent la chaîne de guards dans cet ordre :

```
Request
  → JwtAuthGuard        (token valide ?)
  → TenantGuard         (tenantId présent ? user membre du tenant ?)
  → RoleGuard           (rôle suffisant pour cette action ?)
  → FiscalYearGuard     (fiscalYearId valide si requis ? année non clôturée ?)
  → Controller Handler
```

Un guard qui échoue rejette la requête immédiatement avec l'erreur appropriée. Le handler ne s'exécute jamais si un guard est en échec.

## Contexte tenant

Le contexte tenant est injecté une fois par requête et disponible via décorateur :

```typescript
@Get('invoices')
findAll(@CurrentTenant() tenant: TenantContext) {
  return this.invoicingService.findAll(tenant.id, tenant.fiscalYearId);
}
```

Le `TenantContext` contient : `tenantId`, `tenantProfile`, `userId`, `userRole`, `fiscalYearId`.

## Modules — responsabilités

| Module | Responsabilité |
|---|---|
| `AuthModule` | JWT, sessions, refresh tokens, MFA — *stratégie exacte à définir* |
| `TenancyModule` | CRUD tenant, memberships, invitations, first-user-becomes-admin |
| `UsersModule` | Profils utilisateurs, préférences, gestion des comptes |
| `FiscalYearModule` | Ouverture, clôture, verrouillage des années fiscales |
| `InvoicingModule` | Génération, signature, transmission DGI, archivage |
| `FiduciaireModule` | Workflow d'invitation, gestion des relations, scopes d'accès |
| `ComplianceModule` | Contrôles réglementaires, validation de conformité |
| `AdminModule` | Console admin interne, séparé avec son propre guard d'admin |

## Conventions de code NestJS

- **DTOs** : toutes les entrées utilisent `class-validator`. Pas de validation manuelle dans les services.
- **Repositories** : encapsulent tout accès aux données. Les services ne connaissent pas le moteur de persistance.
- **Services** : contiennent la logique métier uniquement. Pas d'accès direct à la base de données.
- **Transactions** : les opérations multi-entités utilisent des transactions explicites.
- **Erreurs** : les exceptions métier utilisent des classes dédiées (ex. `FiscalYearClosedException`, `TenantAccessDeniedException`).

## Gestion de la multi-tenancy

- Chaque requête de base de données est filtrée par `tenantId`. Un service ne peut pas requêter des données d'un autre tenant.
- Les repositories prennent `tenantId` comme paramètre obligatoire sur toutes les méthodes de lecture et d'écriture.
- La FiduciaireRelationship est le seul mécanisme permettant un accès cross-tenant, et il est géré dans un service dédié.

## Intégration DGI

Le `DgiGatewayModule` encapsule toute communication avec les systèmes DGI :
- Les mécanismes de signature et de transmission sont isolés dans ce module.
- Le retry, la gestion des rejets DGI, et les accusés de réception y sont gérés.
- Les autres modules n'interagissent avec la DGI qu'à travers l'interface de ce module.
