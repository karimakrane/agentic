# Conformité CNDP — Commission Nationale de contrôle de la Protection des Données à caractère personnel

## Contexte réglementaire

La CNDP est l'autorité marocaine de protection des données personnelles, instituée par la **loi 09-08** relative à la protection des personnes physiques à l'égard du traitement des données à caractère personnel. La plateforme de facturation traite des données personnelles d'entreprises et de leurs représentants légaux.

## Périmètre de conformité

### Registre des traitements
- Inventaire de tous les traitements de données personnelles
- Finalité, base légale, catégories de données, durées de conservation
- Destinataires et éventuels transferts hors Maroc

### Catégories de données personnelles traitées
- Données d'identification fiscale : NIF, ICE, CIN des représentants légaux
- Données de contact : nom, prénom, adresse, email, téléphone
- Données contractuelles : coordonnées bancaires, conditions de paiement
- Données de navigation et d'usage de la plateforme

### Droits des personnes concernées
- Droit d'accès aux données personnelles
- Droit de rectification
- Droit d'opposition
- Procédures de traitement des demandes et délais de réponse

### Déclarations CNDP
- Traitements soumis à déclaration préalable
- Traitements soumis à autorisation
- Suivi des déclarations et autorisations obtenues

### Transferts de données hors Maroc
- Identification des flux de données transfrontaliers
- Garanties appropriées pour les transferts vers des pays sans niveau de protection adéquat

## Structure du dossier

```
cndp/
├── README.md                    # Ce fichier
├── register/                    # Registre des traitements
├── pia/                         # Analyses d'impact sur la vie privée (PIA)
├── declarations/                # Déclarations et autorisations CNDP
└── procedures/                  # Procédures droits des personnes
```

## Statut de conformité

> Le registre des traitements et les PIA sont constitués en parallèle du développement des fonctionnalités.
