# Monorepo Architecture

## Décision

e-fatourati est structuré comme un **monorepo**. Ce choix permet de partager des types, des schémas de validation, et des constantes entre le frontend et le backend sans duplication ni désynchronisation.

**Outillage** : non encore décidé (Nx, Turborepo, pnpm workspaces natif). Ne pas présumer d'un outil avant décision formelle.

## Structure cible

```
e-fatourati/
├── apps/
│   ├── web/                  # Next.js — interface utilisateur tenant
│   └── api/                  # NestJS — API backend
│
├── packages/
│   ├── types/                # Types TypeScript partagés (entités, DTOs)
│   ├── schemas/              # Schémas de validation Zod partagés
│   └── utils/                # Utilitaires purs sans effet de bord
│
├── docs/                     # Documentation (ce dossier)
├── compliance/               # Compliance tracking opérationnel
├── threat-model/             # Modèle de menaces
│
├── package.json              # Workspace root
├── tsconfig.base.json        # Config TypeScript partagée
├── .eslintrc.js              # Config ESLint partagée
└── CLAUDE.md                 # Instructions permanentes Claude Code
```

## Règles de dépendance entre packages

```
apps/web  ──imports──> packages/types, packages/schemas, packages/utils
apps/api  ──imports──> packages/types, packages/schemas, packages/utils
apps/web  ✗  ne peut pas importer depuis apps/api
apps/api  ✗  ne peut pas importer depuis apps/web
packages/ ✗  ne peut pas importer depuis apps/
```

La communication entre `web` et `api` passe exclusivement par les APIs HTTP.

## Packages partagés — contraintes

Les packages dans `packages/` ne contiennent que :
- **Types TypeScript** : interfaces, enums, types utilitaires
- **Schémas Zod** : validation partagée entre frontend et backend
- **Constantes** : valeurs fixes sans logique (codes erreur, noms de routes, etc.)

Interdit dans les packages partagés :
- Tout import de framework (Next.js, NestJS, Express)
- Tout effet de bord (appels HTTP, accès base de données, logs)
- Toute dépendance sur l'environnement d'exécution

## Configuration partagée

| Fichier | Portée | Description |
|---|---|---|
| `tsconfig.base.json` | Racine | Config TypeScript héritée par tous les packages |
| `.eslintrc.js` | Racine | Règles ESLint communes + surcharges par package |
| `prettier.config.js` | Racine | Format de code uniforme |
| `jest.config.base.js` | Racine | Config Jest de base partagée — `[assumption]` |

## Builds et CI

> `[assumption]` Les détails de la pipeline CI/CD sont à définir. Comportement attendu :

- Les packages sont buildés dans l'ordre de leurs dépendances.
- Un changement dans `packages/types` doit déclencher le rebuild de `apps/web` et `apps/api`.
- Les tests sont exécutés par workspace, avec possibilité de ne tester que les packages affectés par un changement.
