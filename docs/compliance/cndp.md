# CNDP — Protection des Données Personnelles

> Ce document présente le cadre de conformité CNDP (loi 09-08) appliqué à e-fatourati. Les détails opérationnels (registre des traitements, PIA, déclarations) sont dans `compliance/cndp/`.

## Cadre légal

**Loi 09-08** du 18 février 2009 relative à la protection des personnes physiques à l'égard du traitement des données à caractère personnel.

La CNDP est l'autorité chargée du contrôle de l'application de cette loi.

## Données personnelles traitées par e-fatourati

### Données des représentants légaux (personnes physiques)

| Donnée | Contexte | Catégorie |
|---|---|---|
| Nom, prénom | Inscription, profil, factures | PII directe |
| Email | Authentification, notifications | PII directe |
| Téléphone | Contact, 2FA | PII directe |
| CIN (si collectée) | Vérification identité | PII directe |
| Adresse | Coordonnées légales sur factures | PII directe |

### Données fiscales à caractère personnel

| Donnée | Contexte | Catégorie |
|---|---|---|
| NIF (personne physique) | Facturation, déclarations | PII fiscale |
| Données de paiement | Transactions | PII financière |
| IP, logs de connexion | Sécurité, audit | PII indirecte |

> Les données d'entreprises (NIF personne morale, ICE) ne sont pas des données personnelles au sens de la loi 09-08 sauf si elles permettent d'identifier une personne physique.

## Bases légales des traitements

> **Requires official CNDP verification** — Les bases légales exactes et les obligations de déclaration préalable doivent être vérifiées avec un conseil juridique spécialisé.

| Traitement | Base légale probable |
|---|---|
| Données d'inscription | Exécution du contrat |
| Facturation avec données personnelles | Obligation légale (TVA, CGI) |
| Logs de sécurité et d'audit | Intérêt légitime / Obligation légale |
| Cookies et données de navigation | Consentement |

## Obligations spécifiques à la multi-tenancy

La nature multi-tenant de la plateforme crée des obligations spécifiques :

- **Isolation stricte** : les données personnelles d'un tenant ne peuvent pas être accessibles depuis un autre tenant sans FiduciaireRelationship valide.
- **Fiduciaire comme sous-traitant** : quand une fiduciaire accède aux données d'une entreprise cliente, elle agit potentiellement comme sous-traitant au sens de la loi 09-08. Un contrat de traitement des données est potentiellement requis — **requires official CNDP/legal verification**.
- **Admin interne** : les opérateurs e-fatourati qui accèdent aux données tenant pour support sont soumis aux règles d'accès aux données personnelles. Cet accès est journalisé.

## Droits des personnes concernées

La plateforme doit permettre l'exercice des droits suivants :

| Droit | Délai légal | Mécanisme à prévoir |
|---|---|---|
| Accès | **Requires official verification** | Export des données personnelles de l'utilisateur |
| Rectification | **Requires official verification** | Modification des données de profil |
| Opposition | **Requires official verification** | Désactivation de certains traitements |
| Effacement | **Requires official verification** | Limité par les obligations de rétention fiscale |

> L'effacement est limité par les obligations légales de conservation des données fiscales. Un utilisateur ne peut pas demander la suppression de données nécessaires au respect d'obligations légales.

## Déclarations CNDP

> **Requires official CNDP verification** — Les traitements soumis à déclaration préalable ou à autorisation.

Certains traitements peuvent nécessiter une déclaration préalable ou une autorisation de la CNDP avant mise en production. Ce point doit être évalué par un juriste spécialisé avant le lancement.

## Lien avec le suivi opérationnel

- `compliance/cndp/` — registre des traitements, PIA, déclarations
- `.claude/agents/privacy.md` — agent privacy référent CNDP
- `.claude/skills/pii-review/SKILL.md` — skill de revue PII
