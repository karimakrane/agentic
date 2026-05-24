# Compatibilité GDPR — Règlement (UE) 2016/679

## Contexte

Le GDPR s'applique au traitement de données personnelles de ressortissants de l'Union Européenne, indépendamment de la localisation géographique du responsable du traitement. La plateforme peut traiter des données de clients ou partenaires européens via des transactions de facturation internationale.

## Critères de déclenchement

Le GDPR s'applique dès lors que la plateforme :
- Traite des données de personnes physiques résidant dans l'UE
- Offre des biens ou services à des personnes dans l'UE
- Surveille le comportement de personnes dans l'UE

## Périmètre de compatibilité

### Principes fondamentaux (Art. 5)
- **Licéité, loyauté, transparence** : base légale documentée pour chaque traitement
- **Limitation des finalités** : données collectées pour des finalités déterminées et légitimes
- **Minimisation** : données adéquates, pertinentes, limitées au nécessaire
- **Exactitude** : données tenues à jour
- **Limitation de la conservation** : durées de rétention définies et appliquées
- **Intégrité et confidentialité** : sécurité appropriée par conception et par défaut

### Droits des personnes concernées
- Droit d'accès (Art. 15)
- Droit de rectification (Art. 16)
- Droit à l'effacement (Art. 17) — sous réserve des obligations de rétention fiscale
- Droit à la portabilité (Art. 20)
- Droit d'opposition (Art. 21)

### Transferts hors UE (Art. 46, 47, 49)
- Transferts vers le Maroc : décision d'adéquation ou garanties appropriées
- Clauses contractuelles types si nécessaire
- Documentation des garanties pour chaque flux transfrontalier

### Violations de données (Art. 33, 34)
- Notification CNIL (ou autorité compétente) sous 72h
- Notification aux personnes concernées si risque élevé
- Procédure de réponse aux incidents intégrant les obligations GDPR

## Structure du dossier

```
gdpr/
├── README.md                    # Ce fichier
├── applicability.md             # Analyse d'applicabilité et périmètre
├── legal-bases.md               # Registre des bases légales par traitement
├── data-transfers.md            # Flux transfrontaliers et garanties
└── breach-procedure.md          # Procédure de notification de violations
```

## Relation avec la CNDP

La conformité CNDP (loi 09-08) et la compatibilité GDPR sont complémentaires. Les contrôles documentés dans `../cndp/` couvrent en grande partie les exigences GDPR. Ce dossier documente les exigences GDPR spécifiques non couvertes par la loi 09-08.
