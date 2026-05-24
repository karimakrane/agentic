# Conformité DGSSI — Direction Générale de la Sécurité des Systèmes d'Information

## Contexte réglementaire

La DGSSI est l'autorité nationale marocaine de cybersécurité. Elle établit les référentiels de sécurité applicables aux systèmes d'information traitant des données sensibles, dont les systèmes de facturation électronique fiscale.

## Périmètre de conformité

### Algorithmes et primitives cryptographiques
- Liste des algorithmes approuvés par la DGSSI (synchronisée avec les recommandations ANSSI)
- Tailles de clé minimales requises : RSA ≥ 2048 bits, AES 256 bits, courbes NIST P-256/P-384
- Algorithmes de hachage : SHA-256 minimum
- Protocoles TLS : version 1.3 requise, 1.2 avec configuration restrictive tolérée

### Certification et homologation
- Processus d'homologation DGSSI pour les systèmes traitant des données fiscales
- Qualification des produits de sécurité utilisés
- Certification des autorités de certification émettrices de certificats qualifiés

### Gestion des incidents de sécurité
- Obligation de notification DGSSI pour les incidents significatifs
- Délais de notification réglementaires
- Contenu requis des notifications

### Infrastructure de confiance
- Exigences sur les HSM (Hardware Security Modules)
- Conditions d'utilisation des services cloud
- Exigences de localisation des données

## Structure du dossier

```
dgssi/
├── README.md                    # Ce fichier
├── approved-algorithms.md       # Registre des algorithmes approuvés
├── controls/                    # Contrôles de sécurité DGSSI et statut
└── incidents/                   # Procédures de notification d'incidents
```

## Statut de conformité

> Les documents de cartographie détaillée sont créés au fur et à mesure des validations DGSSI.
