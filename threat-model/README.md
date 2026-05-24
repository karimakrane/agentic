# Threat Model

Ce dossier contient le modèle de menaces de la plateforme de facturation électronique marocaine. Il est maintenu par l'agent `threat-modeling` et mis à jour à chaque changement architectural significatif.

## Méthodologie

- **STRIDE** pour l'identification systématique des menaces par catégorie
- **PASTA** (Process for Attack Simulation and Threat Analysis) pour l'analyse orientée attaquant
- **CVSS v3.1** pour la notation de sévérité des vulnérabilités identifiées

## Actifs critiques

| Actif | Criticité | Description |
|---|---|---|
| Clés de signature électronique | Critique | Clés privées des certificats qualifiés pour la signature des factures |
| Journaux d'audit fiscaux | Critique | Journaux immuables de toutes les transactions fiscales |
| Données de transmission DGI | Élevée | Flux de communication vers les systèmes DGI |
| Données personnelles des émetteurs | Élevée | NIF, ICE, coordonnées des représentants légaux |
| Archives fiscales | Élevée | Factures archivées avec obligation légale de rétention |
| Tokens d'authentification DGI | Élevée | Credentials d'accès aux APIs DGI |

## Frontières de confiance

```
[Internet / Clients] ──TLS 1.3──> [API Gateway]
                                        │
                              [DMZ / Load Balancer]
                                        │
                          [Application Layer] ──mTLS──> [DGI Systems]
                                        │
                    [Domain Layer] ──── [HSM / Key Management]
                                        │
                          [Data Layer] ──chiffré──> [Archive Storage]
```

## Structure du dossier

```
threat-model/
├── README.md                    # Ce fichier — vue d'ensemble
├── dfd/                         # Data Flow Diagrams par composant
│   ├── global-dfd.md            # DFD global de la plateforme
│   ├── invoice-flow.md          # Flux de génération et signature
│   └── dgi-gateway.md          # Flux de transmission DGI
├── stride/                      # Analyse STRIDE par composant
│   ├── api-gateway.md
│   ├── signature-service.md
│   ├── dgi-gateway.md
│   └── archive-service.md
├── risk-register.md             # Registre des menaces et risques résiduels
└── decisions/                   # Décisions de traitement des risques
```

## Processus de mise à jour

1. Tout changement architectural déclenche une revue de la section concernée du threat model.
2. L'agent `threat-modeling` valide la mise à jour via la skill `threat-modeling`.
3. Les nouvelles menaces identifiées sont enregistrées dans `risk-register.md` avec leur statut de traitement.
4. Les risques acceptés formellement sont documentés dans `decisions/` avec justification et approbation.

## Cadence de revue

- **Revue complète** : annuelle ou lors d'un changement d'architecture majeur
- **Revue partielle** : à chaque nouvelle surface d'attaque exposée
- **Revue d'urgence** : après tout incident de sécurité ou publication de CVE impactant les composants utilisés
