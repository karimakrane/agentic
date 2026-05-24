# Infrastructure — Docker

## Décision confirmée

La plateforme utilise **Docker** pour la conteneurisation de ses services. Docker Compose est utilisé pour l'environnement de développement local.

L'infrastructure de déploiement en production **n'est pas encore finalisée**. Ce document décrit uniquement ce qui est confirmé.

## Services identifiés

Les services suivants sont attendus dans la composition Docker. Les technologies non confirmées sont marquées `[TBD]`.

| Service | Image | Statut |
|---|---|---|
| `api` | Build depuis `apps/api/` | Confirmé |
| `web` | Build depuis `apps/web/` | Confirmé |
| `db` | PostgreSQL / MySQL / autre | **TBD** |
| `cache` | Redis / autre | **TBD** |
| `queue` | RabbitMQ / BullMQ / autre | **TBD** |
| `storage` | MinIO / S3-compatible / autre | **TBD** |

Ne pas écrire de `docker-compose.yml` référençant des services `[TBD]` sans décision formelle.

## Environnement de développement local

### Attendus confirmés

- Un seul fichier `docker-compose.yml` à la racine du monorepo pour lancer l'environnement complet.
- Le hot-reload est activé pour `apps/web` et `apps/api` en développement.
- Les variables d'environnement de développement sont dans `.env.local` (non commité).
- Un fichier `.env.example` documente toutes les variables requises avec des valeurs fictives.

### Volumes

- Les données de la base de données sont persistées dans un volume Docker nommé pour éviter la perte entre redémarrages.
- Le code source est monté en volume pour le hot-reload — applicable uniquement en développement.

## Variables d'environnement

Les variables d'environnement sont classées par catégorie. Aucune valeur sensible n'est committée.

| Catégorie | Exemples de variables | Notes |
|---|---|---|
| Application | `NODE_ENV`, `PORT`, `APP_URL` | |
| Base de données | `DATABASE_URL` | Format selon technologie retenue |
| Auth | `JWT_SECRET`, `JWT_EXPIRY` | Jamais committés |
| DGI | `DGI_API_URL`, `DGI_CERT_PATH` | **Requires official DGI verification** |
| Stockage | `STORAGE_ENDPOINT`, `STORAGE_BUCKET` | Selon technologie retenue |
| Email | `SMTP_HOST`, `SMTP_PORT` | Technologie email TBD |

## Décisions en attente — infrastructure production

Les éléments suivants ne sont pas encore décidés et ne doivent pas être assumés :

- Orchestration production (Docker Swarm, Kubernetes, ECS, autre)
- Cloud provider
- Stratégie de scaling horizontal
- Reverse proxy / load balancer (Nginx, Traefik, autre)
- Gestion des certificats TLS en production
- Registry d'images Docker
- Stratégie de backup et de disaster recovery

Ces décisions auront un impact sur l'architecture de déploiement et seront documentées dans `docs/decisions/` sous forme d'ADR lors de leur finalisation.
