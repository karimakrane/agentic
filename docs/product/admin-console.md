# Admin Console

## Définition

La **console d'administration interne** est une application réservée aux opérateurs de e-fatourati. Elle permet de gérer la plateforme elle-même : tenants, utilisateurs, configuration système, santé de la plateforme.

Elle est **entièrement distincte** de l'interface tenant. Un tenant admin n'a pas accès à la console interne, et un admin interne n'a pas de rôle dans les tenants.

## Séparation technique

| Dimension | Interface tenant | Console admin interne |
|---|---|---|
| Authentification | Comptes utilisateurs tenant | Comptes opérateurs distincts |
| Sessions | Scopées tenant + année fiscale | Scopées rôle admin uniquement |
| API | API principale e-fatourati | API admin dédiée (endpoint séparé) |
| Journaux d'audit | Journal tenant | Journal admin distinct |
| Accès réseau | Internet | `[assumption]` Accès restreint (VPN / IP whitelist) |

## Responsabilités de la console admin

### Gestion des tenants
- Voir la liste de tous les tenants (profil, statut, date de création)
- Activer, suspendre, ou clore un tenant
- Consulter les informations légales d'un tenant (NIF, ICE) pour support ou vérification
- Accéder en lecture aux données d'un tenant pour support client — *avec journalisation systématique*

### Gestion des utilisateurs
- Consulter les comptes utilisateurs (sans accès aux mots de passe)
- Forcer la réinitialisation du mot de passe
- Suspendre ou désactiver un compte utilisateur

### Configuration système
- Gestion des paramètres globaux de la plateforme
- Gestion des intégrations DGI (endpoints, certificats, credentials)
- Configuration des notifications système

### Monitoring et santé
- Vue de l'état des services
- Métriques de transmission DGI (succès, échecs, latence)
- Alertes sur les erreurs critiques

### Facturation et abonnements
> `[assumption]` La gestion des abonnements et de la facturation de la plateforme elle-même est à définir.

## Audit trail admin

Toutes les actions effectuées depuis la console admin sont journalisées avec :
- Identité de l'opérateur
- Action effectuée
- Ressource cible (tenant ID, user ID, etc.)
- Horodatage
- Adresse IP

Ce journal est conservé séparément des journaux tenant et ne peut pas être supprimé depuis la console admin elle-même.

## Contraintes de sécurité

- L'accès à la console admin requiert une authentification forte (MFA obligatoire).
- Les actions sensibles (accès aux données d'un tenant, suspension de compte) requièrent une confirmation explicite.
- La console admin n'expose jamais de données personnelles au-delà du strict nécessaire au support.
- Tout accès aux données d'un tenant dans un contexte de support est notifié au tenant admin. `[assumption : à confirmer selon politique]`
