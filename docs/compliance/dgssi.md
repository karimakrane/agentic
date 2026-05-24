# DGSSI — Sécurité des Systèmes d'Information

> **Requires official DGSSI verification** — Ce document synthétise les exigences de sécurité applicables. Les référentiels DGSSI officiels et leurs versions en vigueur doivent être consultés directement pour toute décision d'implémentation.

## Contexte

La Direction Générale de la Sécurité des Systèmes d'Information est l'autorité nationale marocaine de cybersécurité. Elle publie des référentiels techniques et des recommandations applicables aux systèmes d'information traitant des données sensibles.

## Algorithmes cryptographiques

> **Requires official DGSSI verification** — La liste des algorithmes approuvés est définie par la DGSSI et s'aligne généralement sur les recommandations de l'ANSSI française. Les seuils et algorithmes ci-dessous sont indicatifs.

### Signature électronique
- RSA avec padding PKCS#1 v2.1 (RSASSA-PSS) : clé ≥ 2048 bits
- ECDSA sur courbes NIST P-256 ou P-384
- Algorithme de hachage : SHA-256 minimum — SHA-1 et MD5 interdits

### Chiffrement symétrique
- AES 256 bits en mode GCM ou CCM
- Pas de DES, 3DES, RC4

### Protocoles de transport
- TLS 1.3 requis
- TLS 1.2 toléré avec configuration restrictive (cipher suites sécurisées uniquement)
- SSL, TLS 1.0, TLS 1.1 interdits

### Gestion des clés
- Les clés privées de signature sont stockées dans un HSM ou un service équivalent certifié
- Rotation des clés selon politique documentée

## Notification d'incidents

> **Requires official DGSSI verification** — Les obligations de notification et leurs délais exacts.

Les systèmes d'information critiques (dont les systèmes de facturation fiscale) peuvent être soumis à des obligations de notification d'incidents de sécurité à la DGSSI. Les seuils déclencheurs, les délais, et le contenu requis des notifications doivent être vérifiés officiellement.

## Homologation

> **Requires official DGSSI verification** — Vérifier si e-fatourati entre dans le périmètre des systèmes nécessitant une homologation DGSSI.

Certains systèmes d'information traitant des données sensibles peuvent nécessiter une homologation ou une qualification par la DGSSI avant mise en production.

## Implication pour e-fatourati

| Domaine | Application à la plateforme |
|---|---|
| Algorithmes de signature des factures | Doivent être conformes à la liste DGSSI approuvée |
| Chiffrement des données fiscales | AES-256 GCM pour les données au repos et en transit |
| TLS | TLS 1.3 pour toutes les communications (API, DGI gateway, web) |
| HSM | Clés de signature qualifiées dans un HSM certifié |
| Incidents | Procédure de notification DGSSI à définir selon obligations |

## Lien avec le suivi opérationnel

- `compliance/dgssi/` — contrôles opérationnels et statut de conformité
- `.claude/agents/security.md` — agent de sécurité référent DGSSI
- `.claude/skills/cryptographic-review/SKILL.md` — skill de revue cryptographique
