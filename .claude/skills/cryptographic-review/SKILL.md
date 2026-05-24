# Purpose

Évaluer la robustesse, la conformité et la correcte implémentation des mécanismes cryptographiques de la plateforme de facturation électronique marocaine : signature électronique des factures, chiffrement des données, gestion des certificats et des clés.

# Trigger Conditions

- Tout changement dans les algorithmes de signature ou de chiffrement.
- Tout changement dans la gestion des clés cryptographiques ou des certificats.
- Toute intégration d'une nouvelle bibliothèque cryptographique.
- Renouvellement de certificats qualifiés.
- Demande explicite des agents `security` ou `threat-modeling`.
- Revue annuelle des algorithmes en usage.

# Execution Steps

1. **Inventorier les primitives cryptographiques** :
   - Algorithmes de signature : RSA, ECDSA, EdDSA — taille de clé, courbe, padding
   - Algorithmes de hachage : SHA-256 minimum, SHA-1 et MD5 interdits
   - Algorithmes de chiffrement symétrique : AES-256 GCM/CCM
   - Protocoles de transport : TLS 1.3 requis, TLS 1.2 accepté avec configuration stricte
2. **Vérifier la conformité DGSSI** :
   - Les algorithmes sont-ils dans la liste des algorithmes recommandés DGSSI en vigueur ?
   - Les certificats utilisés sont-ils qualifiés et émis par une AC reconnue DGSSI ?
   - Les tailles de clé respectent-elles les minima réglementaires ?
3. **Analyser l'implémentation** :
   - Pas de construction artisanale de primitives cryptographiques — bibliothèques éprouvées uniquement
   - Vecteurs d'initialisation (IV/nonce) aléatoires et non réutilisés
   - Pas d'oracle de padding exposé
   - Gestion correcte des erreurs cryptographiques (pas de timing side-channel)
4. **Vérifier la gestion des clés** :
   - Clés privées exclusivement dans HSM ou service de gestion de clés certifié
   - Procédure de rotation des clés documentée et testée
   - Pas de clé hardcodée, même pour les tests
   - Séparation des clés par environnement (dev ≠ staging ≠ prod)
5. **Vérifier la non-répudiation** :
   - Les signatures de factures permettent-elles la vérification sans accès au système émetteur ?
   - Les horodatages sont-ils conformes aux exigences DGI ?
6. **Produire le rapport** — Algorithmes évalués, conformité DGSSI, vulnérabilités détectées, recommandations, risque résiduel.

# Success Criteria

- Tous les algorithmes en usage sont conformes aux recommandations DGSSI actuelles.
- Aucune clé privée accessible en dehors du HSM ou du service de gestion de clés.
- Aucune primitive cryptographique construite manuellement.
- La non-répudiation des signatures de factures est établie et vérifiable.
- Le rapport est archivé dans `docs/control-evidence/`.

# Failure Handling

- Algorithme déprécié détecté en production → blocage immédiat du déploiement + plan de migration sous 5 jours ouvrés.
- Clé privée exposée hors HSM → révocation immédiate + rotation + analyse d'impact + notification DGSSI.
- Implémentation cryptographique artisanale détectée → refus de merge + remplacement par bibliothèque approuvée obligatoire.
- Certificat qualifié expiré ou révoqué → gel des opérations de signature + renouvellement d'urgence.
