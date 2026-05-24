# User Roles

## Niveaux de rôle

La plateforme distingue deux domaines d'autorisation entièrement séparés :

1. **Domaine tenant** — rôles des utilisateurs au sein d'un tenant
2. **Domaine admin interne** — rôles des opérateurs de la console d'administration e-fatourati

Ces deux domaines ne partagent ni authentification, ni sessions, ni surfaces d'API.

---

## Rôles tenant

### Tenant Admin

- Premier utilisateur d'un tenant, ou promu par un admin existant.
- Contrôle total sur le tenant : configuration, utilisateurs, abonnements, données.
- Peut inviter, créer, suspendre, ou retirer des utilisateurs du tenant.
- Peut initier ou accepter une FiduciaireRelationship.
- Peut ouvrir ou clôturer une année fiscale.

### Utilisateurs avec rôles

> `[assumption]` La granularité des rôles par profil de tenant est à définir précisément. Les éléments suivants sont des hypothèses de travail à valider.

Les rôles ci-dessous sont des propositions initiales :

| Rôle | Portée | Description |
|---|---|---|
| `admin` | Tenant | Accès complet, gestion des membres |
| `accountant` | Tenant | Accès comptabilité et facturation, pas de gestion des membres |
| `invoicer` | Tenant | Création et gestion des factures uniquement |
| `viewer` | Tenant | Lecture seule sur toutes les données du tenant |

Le modèle de rôles devra être validé par profil de tenant (fiduciaire, auto-entrepreneur, company), car les permissions disponibles diffèrent selon le profil.

---

## Accès fiduciaire sur les tenants gérés

Un utilisateur membre d'un tenant **fiduciaire** peut accéder à des données d'un tenant **company** si :

1. Une `FiduciaireRelationship` active existe entre les deux tenants.
2. L'utilisateur a un rôle dans le tenant fiduciaire lui donnant accès aux clients.
3. Le périmètre d'accès (`grantedScopes`) défini dans la relation autorise l'action demandée.

> `[assumption]` La structure des `grantedScopes` est à définir. Elle doit permettre à une entreprise de contrôler précisément ce à quoi sa fiduciaire a accès.

L'accès fiduciaire est **toujours en lecture par défaut**. L'écriture pour le compte d'un client est un scope supplémentaire à accorder explicitement.

---

## Rôles admin interne

Réservés aux opérateurs de la plateforme e-fatourati. Gèrent la plateforme, pas les données métier des tenants.

> `[assumption]` Les rôles internes exacts sont à définir selon les besoins opérationnels.

Exemples de rôles attendus :

| Rôle | Description |
|---|---|
| `super-admin` | Accès complet à la console d'administration |
| `support` | Lecture des données tenant pour support client, accès restreint |
| `ops` | Gestion de l'infrastructure et des déploiements |

Toutes les actions admin internes sont journalisées dans un audit trail distinct du journal tenant.

---

## Invitations et création de comptes

- Un **tenant admin** peut inviter un utilisateur externe par email → l'utilisateur reçoit un lien d'activation.
- Un **tenant admin** peut créer directement un compte utilisateur pour un membre de son équipe.
- Un utilisateur invité peut rejoindre le tenant sans créer de nouveau compte si un compte existe déjà sous le même email.
