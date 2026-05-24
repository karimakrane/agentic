# GDPR — Compatibilité pour les Données Européennes

## Positionnement

Le GDPR (Règlement (UE) 2016/679) est une réglementation européenne. e-fatourati est une plateforme marocaine opérée sous le droit marocain. Cependant, le GDPR peut s'appliquer dans des cas spécifiques liés aux données de personnes résidant dans l'Union Européenne.

## Analyse d'applicabilité

Le GDPR s'applique à e-fatourati si la plateforme :

1. **Traite des données de personnes physiques résidant dans l'UE**, même si e-fatourati est établi hors UE (critère de l'Art. 3.2).
2. **Offre des biens ou services** à des personnes dans l'UE (ex. entreprises marocaines facturant des clients européens, ou ressortissants européens utilisant la plateforme).
3. **Surveille le comportement** de personnes dans l'UE.

> L'analyse d'applicabilité précise doit être conduite par un juriste. Ce document présente les scénarios probables, pas une conclusion juridique.

## Scénarios d'applicabilité probables

| Scénario | GDPR applicable ? |
|---|---|
| Tenant marocain facturant exclusivement des clients marocains | Probablement non |
| Tenant marocain facturant des clients dans l'UE | Probablement oui pour les données des clients EU |
| Ressortissant européen résidant au Maroc utilisant la plateforme | À analyser selon le contexte |
| Fiduciaire marocaine gérant une entreprise ayant des clients EU | À analyser |

## Complémentarité CNDP / GDPR

La conformité CNDP (loi 09-08) couvre la grande majorité des obligations de protection des données. Le GDPR ajoute des exigences spécifiques dans les domaines suivants :

| Domaine | CNDP (loi 09-08) | GDPR (complément) |
|---|---|---|
| Portabilité des données | Non prévu explicitement | Droit à la portabilité (Art. 20) |
| Notification des violations | **Requires official CNDP verification** | 72h à l'autorité + notification personnes si risque élevé |
| DPO (Délégué à la protection des données) | Non prévu | Potentiellement obligatoire selon volume de traitement |
| PIA obligatoire | Non prévu explicitement | Obligatoire pour traitements à risque élevé (Art. 35) |
| Transferts hors UE | Non applicable | Mécanismes requis (décision d'adéquation, clauses types) |

## Transferts de données hors UE

Si des données de personnes dans l'UE transitent par e-fatourati, le transfert vers le Maroc constitue un transfert hors UE au sens du GDPR.

> **Requires official legal verification** — Le Maroc ne dispose pas (à la date de rédaction) d'une décision d'adéquation de la Commission Européenne. Les transferts doivent s'appuyer sur des garanties appropriées (clauses contractuelles types, règles d'entreprise contraignantes, etc.).

## Obligations GDPR spécifiques à surveiller

Si le GDPR est confirmé applicable après analyse juridique :

- **Registre des traitements** : obligatoire si > 250 employés ou traitements à risque (Art. 30)
- **Bases légales** : chaque traitement doit avoir une base légale GDPR explicite
- **Mentions d'information** : les personnes concernées doivent être informées de leurs droits
- **Exercice des droits** : délai de réponse de 30 jours (vs délai CNDP à vérifier)
- **Violations** : notification autorité compétente dans les 72h

## Recommandation

Avant de proposer la plateforme à des entreprises ayant des clients dans l'UE ou à des résidents européens :

1. Mener une analyse d'applicabilité GDPR avec un juriste.
2. Si applicable, conduire un mapping des écarts entre loi 09-08 et GDPR.
3. Mettre en place les mécanismes de transfert appropriés si nécessaire.

## Lien avec le suivi opérationnel

- `compliance/gdpr/` — analyse d'applicabilité, bases légales, transferts
- `.claude/agents/privacy.md` — agent privacy référent CNDP/GDPR
