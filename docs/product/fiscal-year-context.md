# Fiscal Year Context

## Principe fondamental

L'**année fiscale** est un contexte obligatoire dans e-fatourati. Toute opération comptable, fiscale, ou de reporting est scopée à une année fiscale. Ce n'est pas une feature optionnelle — c'est une contrainte architecturale et réglementaire.

## Pourquoi l'année fiscale est mandatory

1. **Obligation légale** : la comptabilité marocaine exige une séparation stricte des exercices comptables.
2. **Cohérence des données** : agréger des données sans contexte temporel produirait des résultats incorrects ou non auditables.
3. **Conformité DGI** : les déclarations fiscales (TVA, IS, IR) sont périodiques et liées à l'exercice.
4. **Clôture et immuabilité** : un exercice clôturé doit être verrouillé — aucune écriture rétroactive sans procédure explicite.

## Définition de l'année fiscale au Maroc

> **Requires official DGI verification** — les informations suivantes reflètent la pratique générale mais doivent être vérifiées contre le Code Général des Impôts et les circulaires DGI en vigueur.

- L'exercice comptable standard au Maroc est l'année civile : **1er janvier au 31 décembre**.
- Des exercices décalés sont possibles pour certaines catégories d'entreprises, sous conditions.
- Pour les auto-entrepreneurs, le régime simplifié peut avoir des règles spécifiques.

La plateforme doit supporter des années fiscales avec des dates de début et de fin configurables, même si l'année civile sera le cas le plus fréquent.

## Comportement attendu

### Sélection de l'année fiscale active

- À chaque session, l'utilisateur doit avoir une année fiscale active sélectionnée.
- L'interface affiche clairement le contexte actif (tenant + année fiscale).
- Le changement d'année fiscale est une action explicite, pas automatique.

### Création d'une nouvelle année fiscale

- Seul un tenant admin peut ouvrir une nouvelle année fiscale.
- La création d'une nouvelle année ne clôture pas automatiquement la précédente.
- La clôture est une action explicite, irréversible sans procédure dédiée.

### Clôture d'une année fiscale

| Statut | Comportement |
|---|---|
| `open` | Lecture et écriture autorisées |
| `closed` | Lecture autorisée, écriture interdite — clôture comptable effectuée |
| `locked` | Lecture seule, archivage définitif — aucune modification possible |

La transition `open` → `closed` → `locked` est unidirectionnelle.

### Propagation dans l'API

Le `fiscalYearId` doit être :
- Présent dans toutes les requêtes créant des documents financiers
- Validé côté serveur (le fiscalYear doit être `open`, appartenir au tenant courant)
- Présent dans tous les journaux d'audit liés à des opérations financières

## Cas d'usage à couvrir

| Cas | Comportement attendu |
|---|---|
| Consultation d'une année clôturée | Lecture seule, indication visuelle claire |
| Tentative d'écriture sur année clôturée | Erreur explicite rejetée au niveau service |
| Comparaison multi-années | `[assumption]` Requêtes cross-fiscalYear via API dédiée, pas via contexte actif |
| Premier exercice d'un nouveau tenant | Création guidée lors de l'onboarding |
| Rattachement d'une facture à une mauvaise année | Correction via procédure d'avoir ou d'annulation selon règles DGI — **requires official verification** |
