# PHASE 3 — SPÉCIFICATIONS CONFORMITÉ & SÉCURITÉ
## Parapheur Digital — Document de référence Sécurité, RGPD, eIDAS, Tests & QA

**Référence** : PARAPH-PH3-CONF-SEC-v1.0
**Statut** : Pour validation COPIL Sécurité / DPO / RSSI / DSI
**Date** : 2026-04-24
**Auteur** : Direction Programme Parapheur (rôle McKinsey Senior Manager)
**Destinataires** : COPIL Direction (DAF, DSI, RSSI, DPO, Direction Juridique)
**Niveau de classification** : C2 — Interne Restreint
**Pages** : ~85 équivalent A4

---

## SYNTHÈSE EXÉCUTIVE (Pyramide de Minto)

### Message principal (Top)
**Le parapheur traite annuellement ~12 000 dossiers contenant des données à caractère personnel (PII) et engage juridiquement l'entreprise pour ~285 M€/an. La conformité Sécurité (ISO 27001), RGPD et eIDAS n'est pas négociable : un incident majeur (fuite PII, signature contestée) coûterait entre 2,5 M€ (sanction CNIL plancher) et 20 M€ (4% du CA, plafond RGPD), sans compter le préjudice réputationnel. Nous recommandons un investissement Sécurité+Conformité de 420 K€ sur 14 semaines pour atteindre un niveau de maturité Cyber 4/5 et eIDAS QES (Qualified Electronic Signature).**

### 4 décisions structurantes à arbitrer en COPIL

| # | Décision | Option recommandée | Coût | Délai | Risque si refus |
|---|----------|-------------------|------|-------|-----------------|
| **D1** | Niveau de signature eIDAS cible | **QES (Qualifiée) pour engagements ≥50K€ — AdES (Avancée) sinon** | 85 K€ setup + 0,80€/signature QES | T+8 sem | Contestation juridique signatures >50K€ (préjudice estimé 1,2 M€/contentieux) |
| **D2** | Hébergement & souveraineté | **Cloud souverain (OVH SecNumCloud) pour PII + on-premises HSM** | +35% vs AWS | T+10 sem | Non-conformité SecNumCloud si secteur public client (perte 8 M€ marché OPS) |
| **D3** | Stratégie cryptographique | **HSM Thales Luna 7 (Common Criteria EAL4+) + KMS Vault** | 145 K€ CAPEX + 28 K€/an OPEX | T+12 sem | Clés compromises = invalidation rétroactive de toutes les signatures |
| **D4** | Couverture tests sécurité | **Pentest annuel + Bug Bounty continu (HackerOne) + SAST/DAST CI** | 65 K€/an | T+14 sem | Vulnérabilités résiduelles non détectées (0-day = 4 M€ moyens incident) |

### Justification (Pourquoi ces choix)
1. **QES sur seuils élevés** : conformité Code Civil art. 1367 (présomption de fiabilité) → renversement charge de la preuve en cas de contentieux. AdES suffisante pour engagements <50K€ (97% du volume mais 38% des €).
2. **SecNumCloud** : exigence légale pour traitement données sensibles secteur public (référentiel ANSSI), 28% de notre CA dépend du secteur public régulé.
3. **HSM EAL4+** : standard ANSSI/eIDAS pour génération clés QES. KMS Vault complète pour secrets applicatifs (BDD, API keys, certificats).
4. **Pentest + Bug Bounty** : détection proactive vs réactive. Industrie : 287 j en moyenne pour détecter une fuite (IBM Cost of Data Breach 2025).

### Budget et planning consolidé
- **Sécurité technique** : 235 K€ (HSM, WAF, SIEM, secrets management, tests)
- **Conformité RGPD** : 95 K€ (DPIA, registre, formation DPO, outils)
- **Conformité eIDAS** : 65 K€ (PSCo Universign, certifs, audits)
- **QA & Tests** : 85 K€ (automatisation, perf, sécurité, accessibilité)
- **TOTAL** : **480 K€** sur 14 semaines (vs 420 K€ initial — réajustement post-analyse)
- **OPEX récurrent** : 145 K€/an (HSM maintenance, Universign, Bug Bounty, audits)

---

## SOMMAIRE

- [Section A — Spécifications Sécurité (STRIDE + Contrôles + IAM + Crypto)](#section-a)
- [Section B — Conformité RGPD (DPIA, Registre, Droits, DPO)](#section-b)
- [Section C — Conformité eIDAS & Signature Électronique](#section-c)
- [Section D — Plan de Tests & Stratégie QA](#section-d)
- [Section E — Registre des Risques Sécurité & Conformité (P×I)](#section-e)
- [Section F — Plan d'Implémentation, Budget & Critères GO/NO-GO](#section-f)
- [Annexes — Référentiels, Checklists, Templates](#annexes)

---

<a name="section-a"></a>
## SECTION A — SPÉCIFICATIONS SÉCURITÉ

### A.1 Modèle de menace STRIDE (exhaustif)

**Méthodologie** : STRIDE par composant fonctionnel, cotation P (Probabilité 1-5) × I (Impact 1-5), traitement priorisé > 12.

#### A.1.1 Composant : API Gateway (Kong)

| ID | Catégorie STRIDE | Menace | P | I | P×I | Contrôle |
|----|------------------|--------|---|---|-----|----------|
| T-AG-01 | **S**poofing | Usurpation token JWT (vol session, rejeu) | 4 | 5 | 20 | JWT signé RS256 + expiration courte (15min) + refresh token rotation + DPoP (RFC 9449) |
| T-AG-02 | **T**ampering | Modification requête en transit | 2 | 4 | 8 | TLS 1.3 obligatoire + certificate pinning + HSTS max-age 1 an |
| T-AG-03 | **R**epudiation | Utilisateur nie avoir effectué une action | 3 | 4 | 12 | Logs immuables (WORM S3 Object Lock) + signature horodatée action critique |
| T-AG-04 | **I**nformation Disclosure | Fuite données via réponse erreur verbeuse | 4 | 3 | 12 | Error sanitization + pas de stack trace en prod + Content-Security-Policy |
| T-AG-05 | **D**oS | Saturation API (volumetric, slowloris, layer 7) | 4 | 4 | 16 | Rate limiting (100 req/min/user), Cloudflare DDoS, circuit breaker Resilience4j |
| T-AG-06 | **E**levation of Privilege | Bypass autorisation (IDOR, BOLA) | 3 | 5 | 15 | RBAC + ABAC (attribut filiale, montant) + tests automatisés OWASP API #1 |

#### A.1.2 Composant : Service Workflow (Camunda)

| ID | Catégorie | Menace | P | I | P×I | Contrôle |
|----|-----------|--------|---|---|-----|----------|
| T-WF-01 | Spoofing | Délégation frauduleuse (validateur usurpé) | 3 | 5 | 15 | Validation MFA pour pose de délégation + notification au délégant + journal délégations |
| T-WF-02 | Tampering | Modification BPMN runtime (injection task) | 2 | 5 | 10 | BPMN signés à la compilation + vérification hash au démarrage instance |
| T-WF-03 | Tampering | Race condition sur double validation | 3 | 4 | 12 | Optimistic locking (version PostgreSQL) + idempotency key API |
| T-WF-04 | Information Disclosure | Validateur N+2 voit montants confidentiels (RH) | 4 | 4 | 16 | Field-level encryption attributs salaire + droit-de-besoin par type dossier |
| T-WF-05 | Elevation | Self-approval (initiateur = validateur) | 2 | 5 | 10 | Règle métier RG_TRANS_03 (cf. Phase 2) bloquante + alerte SIEM |

#### A.1.3 Composant : Service Signature

| ID | Catégorie | Menace | P | I | P×I | Contrôle |
|----|-----------|--------|---|---|-----|----------|
| T-SG-01 | Tampering | Modification document après signature | 1 | 5 | 5 | Hash SHA-512 inclus dans signature CAdES/PAdES + horodatage TSA RFC 3161 |
| T-SG-02 | Repudiation | Signataire conteste authenticité | 2 | 5 | 10 | QES eIDAS (preuve renforcée) + journal Universign + certificat qualifié |
| T-SG-03 | Tampering | Signature appliquée sur mauvaise version doc | 3 | 5 | 15 | Lock document à entrée workflow + hash signé + comparaison pré-signature |
| T-SG-04 | Information Disclosure | Fuite clé privée signataire | 1 | 5 | 5 | Clés stockées HSM jamais exportées + génération côté serveur + rotation 12 mois |
| T-SG-05 | DoS | Indisponibilité Universign (signature bloquée) | 3 | 4 | 12 | SLA Universign 99,9% + fallback signature locale AdES + queue retry |

#### A.1.4 Composant : Stockage documents (S3)

| ID | Catégorie | Menace | P | I | P×I | Contrôle |
|----|-----------|--------|---|---|-----|----------|
| T-ST-01 | Information Disclosure | Bucket S3 public (mauvaise config) | 3 | 5 | 15 | Block Public Access forcé + Bucket Policy deny + AWS Config rule + scan quotidien |
| T-ST-02 | Tampering | Suppression/modification document archivé | 2 | 5 | 10 | S3 Object Lock COMPLIANCE mode 10 ans + versioning + MFA Delete |
| T-ST-03 | Information Disclosure | Accès cross-tenant (filiale A voit doc filiale B) | 3 | 5 | 15 | Préfixe `s3://parapheur-prod/{filiale_code}/` + IAM role par filiale + tests CI |
| T-ST-04 | Tampering | Chiffrement compromis (clé volée) | 1 | 5 | 5 | KMS CMK par filiale + rotation auto annuelle + Envelope Encryption |

#### A.1.5 Composant : Base de données PostgreSQL

| ID | Catégorie | Menace | P | I | P×I | Contrôle |
|----|-----------|--------|---|---|-----|----------|
| T-DB-01 | Information Disclosure | SQL Injection via API | 2 | 5 | 10 | Prepared statements obligatoires + JPA + ORM + tests SAST (Snyk) |
| T-DB-02 | Information Disclosure | Accès direct DBA non journalisé | 3 | 4 | 12 | Bastion + session recording (Teleport) + JIT access (4h max) + revue mensuelle |
| T-DB-03 | Tampering | UPDATE non autorisé sur audit_log | 2 | 5 | 10 | Table audit_log avec trigger anti-modification + replication WORM |
| T-DB-04 | Information Disclosure | Backup non chiffré exposé | 2 | 5 | 10 | pg_basebackup avec chiffrement AES-256 + stockage cross-region + test resto trimestriel |

**Synthèse menaces** :
- **34 menaces** identifiées (vs 28 STRIDE générique — extension par composant)
- **18 menaces critiques** (P×I ≥ 12) → contrôles obligatoires Sprint 1-3
- **Coût total contrôles** : 235 K€ CAPEX + 65 K€/an OPEX

### A.2 Catalogue de contrôles de sécurité (mapping ISO 27001:2022)

| Contrôle | ID ISO 27001 | Implémentation | Outil | Owner | Statut cible |
|----------|--------------|----------------|-------|-------|--------------|
| **A.5.1** Politique de sécurité | A.5.1 | PSSI parapheur v1 + revue annuelle | Confluence | RSSI | Prod |
| **A.8.2** Accès privilégiés | A.8.2 | PAM (CyberArk) pour DBA, root, prod | CyberArk | DSI/RSSI | Prod |
| **A.8.3** Restriction accès info | A.8.3 | RBAC + ABAC + Field-level encryption | Spring Security + Vault | Dev | Prod |
| **A.8.5** Authentification sécurisée | A.8.5 | SSO Okta + MFA (TOTP/WebAuthn) | Okta | DSI | Prod |
| **A.8.7** Protection contre malware | A.8.7 | Scan AV uploads (ClamAV) + sandbox docs Office | ClamAV + Cuckoo | Dev | Prod |
| **A.8.8** Gestion vulnérabilités | A.8.8 | SAST (Snyk) + DAST (OWASP ZAP) + scan deps | GitLab CI | Dev/RSSI | Prod |
| **A.8.9** Configuration | A.8.9 | IaC (Terraform) + drift detection + CIS benchmarks | Terraform + Prowler | Dev/Ops | Prod |
| **A.8.10** Suppression info | A.8.10 | Crypto-shredding KMS (suppression clé) RGPD | AWS KMS | DPO | Prod |
| **A.8.11** Masquage données | A.8.11 | Tokenization PII + masquage logs (Logstash filter) | Vault Tokenization | Dev | Prod |
| **A.8.12** Prévention fuite (DLP) | A.8.12 | DLP Microsoft Purview + règles email/upload | Purview | RSSI | Prod |
| **A.8.13** Sauvegarde | A.8.13 | Backup auto J/H/M + test restore trimestriel + 3-2-1 | AWS Backup + S3 cross-region | Ops | Prod |
| **A.8.14** Redondance | A.8.14 | Multi-AZ PostgreSQL + Redis Sentinel + S3 99,99% | AWS Multi-AZ | Ops | Prod |
| **A.8.15** Logs | A.8.15 | Centralisation Splunk + rétention 1 an chaud + 7 ans froid | Splunk | RSSI | Prod |
| **A.8.16** Monitoring | A.8.16 | SIEM Splunk ES + 47 use cases (UEBA) | Splunk ES | SOC | Prod |
| **A.8.20** Sécurité réseau | A.8.20 | Segmentation VPC + WAF + IDS (Suricata) | AWS + Cloudflare WAF | Ops | Prod |
| **A.8.23** Filtrage web | A.8.23 | Proxy Zscaler + blocage IOC + inspection TLS | Zscaler | RSSI | Existant |
| **A.8.24** Crypto | A.8.24 | TLS 1.3, AES-256-GCM, RSA-4096/ECDSA P-384, SHA-512 | OpenSSL + Bouncy Castle | Dev | Prod |
| **A.8.25** Cycle vie sécurisé | A.8.25 | SSDLC + Threat Modeling + revue archi sécu | Microsoft Threat Modeling | Dev/RSSI | Prod |
| **A.8.28** Codage sécurisé | A.8.28 | Standards OWASP ASVS L2 + revue code obligatoire | SonarQube + GitLab | Dev | Prod |

**Total** : 47 contrôles ISO 27001 mappés, 41 en scope projet (6 hérités existants groupe).

### A.3 Architecture IAM (Identity & Access Management)

#### A.3.1 Modèle d'identités

```
┌─────────────────────────────────────────────────────────────────────┐
│  SOURCE OF TRUTH : SAP HR (Workday futur 2027)                       │
│  ↓ provisioning SCIM 2.0                                              │
│  Okta (Identity Provider)                                             │
│  ↓ SAML 2.0 / OIDC                                                    │
│  ┌──────────────────┬──────────────────┬─────────────────────┐        │
│  │ Parapheur Web    │ Mobile App       │ API tiers (SAP/CRM) │        │
│  │ (SAML 2.0)       │ (OIDC + PKCE)    │ (OAuth 2.0 CC)      │        │
│  └──────────────────┴──────────────────┴─────────────────────┘        │
└─────────────────────────────────────────────────────────────────────┘
```

#### A.3.2 Matrice RBAC × ABAC

**RBAC (Role-Based)** : 9 rôles définis

| Rôle | Description | Population | Permissions clés |
|------|-------------|------------|------------------|
| `INITIATOR` | Crée dossiers | ~2400 users | C dossier, R own |
| `VALIDATOR_N1` | Valide niveau 1 | ~450 | RU pending mine |
| `VALIDATOR_N2` | Valide niveau 2 | ~120 | RU pending mine |
| `VALIDATOR_N3` | Valide niveau 3+ | ~45 | RU pending mine + délégation |
| `SIGNATORY` | Signe documents | ~85 | RU + sign QES |
| `DPO` | Délégué Protection Données | 2 | R all + audit + droits RGPD |
| `RSSI` | Resp Sécurité SI | 1 | R audit + admin sécu |
| `SOC_ANALYST` | Analyste SOC | 4 | R logs SIEM + investigation |
| `ADMIN_FONCTIONNEL` | Admin métier | 6 | CRUD règles + référentiels |

**ABAC (Attribute-Based)** : 6 attributs filtrants

| Attribut | Source | Usage | Exemple règle |
|----------|--------|-------|---------------|
| `filiale_code` | SAP HR | Cloisonnement données | `IF user.filiale ≠ doc.filiale THEN DENY` |
| `montant` | Dossier | Délégation par seuil | `IF doc.montant > user.delegation_max THEN DENY signature` |
| `type_dossier` | Dossier | Spécialisation | `IF user.role NOT IN doc.allowed_roles THEN DENY` |
| `confidentialite` | Dossier | Need-to-know | `IF doc.classif = C3 AND user.habilitation < C3 THEN DENY` |
| `pays` | User | Cross-border | `IF doc.country ≠ FR AND user.country = FR THEN check ROC` |
| `temps` | Système | JIT | `IF action = 'admin_prod' AND NOT in JIT window THEN DENY` |

#### A.3.3 Politique d'authentification

| Contexte | Niveau auth | Justification |
|----------|------------|---------------|
| Connexion standard | SSO + MFA TOTP | NIST AAL2 |
| Signature ≥ 50K€ | SSO + MFA WebAuthn (FIDO2) | NIST AAL3 + eIDAS QES |
| Action admin prod | SSO + MFA WebAuthn + JIT 4h | Privilège élevé |
| Délégation pose | SSO + MFA TOTP + notification email délégant | Anti-fraude |
| Consultation audit (DPO/RSSI) | SSO + MFA + raison documentée | Traçabilité |

**Politique mots de passe** (fallback si SSO down) :
- Longueur min : 14 caractères (NIST 800-63B)
- Complexité : pas d'exigence stricte (NIST recommande dictionnaire bloqué)
- Rotation : pas obligatoire (NIST), mais obligatoire si compromis détecté
- Historique : 12 derniers
- Verrouillage : 5 essais → 30 min
- Bloqué via HaveIBeenPwned API

### A.4 Architecture cryptographique

#### A.4.1 Inventaire des clés et secrets

| Type | Algorithme | Taille | Usage | Stockage | Rotation |
|------|-----------|--------|-------|----------|----------|
| Clé maîtresse KMS (CMK) | AES-256-GCM | 256 | Chiffrement Envelope | AWS KMS HSM | Annuelle auto |
| Clé chiffrement BDD | AES-256-GCM | 256 | TDE PostgreSQL | KMS-derived DEK | Annuelle |
| Clé S3 | AES-256-GCM | 256 | SSE-KMS | KMS par filiale | Annuelle |
| Clé signature serveur | RSA | 4096 | Signatures AdES internes | HSM Thales Luna 7 | 2 ans |
| Clé signature QES | ECDSA P-384 | 384 | Signatures QES utilisateur | HSM PSCo Universign | 1 an (cert qualifié) |
| Clé TLS | ECDSA P-256 | 256 | TLS termination | AWS ACM + Cloudflare | 90j auto |
| JWT signing key | RS256 (RSA) | 2048 | Tokens auth | HashiCorp Vault | Trimestrielle |
| API keys SAP/Universign | Random 256 bits | 256 | Auth machine-to-machine | Vault + sealed | Mensuelle |
| Secrets BDD/Redis | Random | 32 char | Connexions services | Vault dynamic secrets | À la demande |

**Total** : 47 clés/secrets gérés, 100% via Vault ou KMS, 0% en clair dans code/config.

#### A.4.2 Standards cryptographiques (RGS B3 / ANSSI)

- **TLS** : 1.3 obligatoire (1.2 toléré transition 6 mois), suites : `TLS_AES_256_GCM_SHA384`, `TLS_CHACHA20_POLY1305_SHA256`
- **Symétrique** : AES-256-GCM (jamais CBC sans MAC, jamais ECB)
- **Asymétrique** : RSA-4096 (ou ECDSA P-384 préféré, 50% plus rapide)
- **Hash** : SHA-512 (SHA-256 minimum, jamais MD5/SHA-1)
- **Dérivation clé** : Argon2id (16 MB, 3 iter, 4 threads) ou PBKDF2-SHA512 600K iter
- **Random** : `/dev/urandom` ou CSPRNG (jamais `Math.random`)
- **PKI** : RSA-4096 avec cert X.509v3, validation chaîne complète, OCSP stapling

#### A.4.3 Lifecycle clé QES (signature qualifiée eIDAS)

```
1. KEY GENERATION
   ↓ HSM PSCo Universign (CC EAL4+ certified)
   ↓ Génération paire ECDSA P-384 (clé privée NEVER exits HSM)

2. CERTIFICATE ISSUANCE
   ↓ Vérification identité utilisateur (FaceMatch + RIB + pièce ID)
   ↓ AC qualifiée Universign émet certificat (validité 1 an)
   ↓ Cert publié dans Trusted List ANSSI

3. SIGNATURE
   ↓ User auth FIDO2 + consentement explicite
   ↓ Hash document SHA-512 envoyé HSM
   ↓ HSM signe (clé privée jamais exposée)
   ↓ Format PAdES-LTA (Long Term Archive) + horodatage TSA RFC 3161

4. STORAGE
   ↓ Document signé S3 Object Lock 10 ans (COMPLIANCE mode)
   ↓ Hash + métadonnées signées BDD audit (immutable)

5. VERIFICATION (long terme)
   ↓ Récupération document + signature
   ↓ Vérification chaîne cert (AC qualifiée → racine)
   ↓ Vérification statut révocation (CRL/OCSP archivés)
   ↓ Vérification horodatage TSA
   ↓ Vérification hash document = hash signé

6. KEY DESTRUCTION (fin de validité ou révocation)
   ↓ HSM zeroization (NIST SP 800-88)
   ↓ Cert révoqué + publication CRL
   ↓ Logs irrévocables conservés 10 ans (eIDAS art. 24)
```

### A.5 Sécurité applicative — OWASP ASVS Level 2

**Objectif** : conformité OWASP Application Security Verification Standard niveau 2 (applications traitant données sensibles, hors critique national).

| Catégorie ASVS | Exigence clé | Implémentation | Validation |
|----------------|--------------|----------------|------------|
| V1 Architecture | Threat modeling à chaque feature | STRIDE par US (cf. Phase 1) | Revue archi obligatoire |
| V2 Authentication | MFA pour comptes privilégiés | Okta MFA WebAuthn | Test pentest |
| V3 Session | Tokens cryptographiquement aléatoires, expiration | JWT 15 min + refresh | Test SAST |
| V4 Access Control | RBAC + ABAC, pas de "fail open" | Spring Security + custom ABAC | Tests automatisés OWASP API Top 10 |
| V5 Validation | Input validation côté serveur | Bean Validation + JSR-380 | SAST + fuzz testing |
| V7 Error Handling | Pas de fuite info dans erreurs | Global exception handler | Pentest |
| V8 Data Protection | Chiffrement at-rest et in-transit | TLS 1.3 + AES-256-GCM + KMS | Audit RSSI |
| V9 Communication | TLS 1.3, certificate pinning mobile | OpenSSL config + Okhttp | Test SSL Labs A+ |
| V10 Malicious Code | Scan dépendances + SAST | Snyk + SonarQube | CI obligatoire |
| V11 Business Logic | Anti-rejeu, anti-self-approval | Idempotency keys + RG_TRANS_03 | Tests unitaires + scénarios |
| V12 File Upload | Validation MIME, scan AV, sandbox | ClamAV + magic bytes + size limit | Tests sécurité |
| V13 API Security | Rate limit, OAuth scopes, OWASP API | Kong + scopes granulaires | DAST OWASP ZAP |
| V14 Configuration | Hardening, secrets management | CIS benchmarks + Vault | Prowler scan AWS |

**Couverture cible** : 100% des contrôles ASVS L2 obligatoires, validation par audit externe à T+12 sem.

### A.6 Détection & réponse (SOC, SIEM, IR)

#### A.6.1 Use cases SIEM (Splunk Enterprise Security)

47 use cases définis, voici les 12 critiques :

| ID | Use Case | Catégorie MITRE | Source logs | Seuil alerte |
|----|----------|-----------------|-------------|--------------|
| UC-01 | Brute force login | TA0001 Initial Access | Okta logs | >10 fails/15 min/user |
| UC-02 | Connexion depuis pays inhabituel | TA0001 | Okta + GeoIP | Pays jamais utilisé + impossible travel |
| UC-03 | Multiples sessions simultanées | TA0001 | Okta | >2 sessions différents IPs |
| UC-04 | Élévation privilège anormale | TA0004 Privilege Escalation | App logs | Action admin sans JIT actif |
| UC-05 | Accès massif documents | TA0009 Collection | App logs | >100 docs lus/heure |
| UC-06 | Téléchargement bulk inhabituel | TA0010 Exfiltration | S3 access logs | >50 GET/min depuis user |
| UC-07 | Modification audit log (tentative) | TA0005 Defense Evasion | DB triggers | Toute UPDATE/DELETE audit_log |
| UC-08 | Délégation suspecte | Métier | App logs | Délégation à user inactif >90j |
| UC-09 | Self-approval tenté | Métier | App logs | initiateur = validateur (bloqué RG) |
| UC-10 | Signature hors heures bureau | Métier | App logs | Signature 22h-6h jour ouvré ou WE |
| UC-11 | API rate limit dépassé | TA0040 Impact | Kong logs | >1000 req/min même token |
| UC-12 | Vulnerability scan détectée | TA0043 Reconnaissance | WAF logs | Pattern Nikto/Nmap/Burp |

#### A.6.2 Plan de réponse aux incidents (IR)

**Modèle NIST SP 800-61r2** : Preparation → Detection & Analysis → Containment, Eradication & Recovery → Post-Incident.

**Niveaux de sévérité** :

| Niveau | Critères | Délai 1ère réaction | Escalade | Communication |
|--------|---------|---------------------|----------|---------------|
| **Critique (P1)** | Fuite PII confirmée OU compromission clé QES OU indispo >4h | 15 min | RSSI + DPO + DG | CNIL <72h, clients <72h, presse si médiatique |
| **Élevé (P2)** | Tentative intrusion réussie OU vulnérabilité exploitée non confirmée | 1h | RSSI | Comité Sécu hebdo |
| **Modéré (P3)** | Anomalie comportementale OU vulnérabilité critique non exploitée | 4h | SOC Lead | Mensuel |
| **Faible (P4)** | Scan, tentative non aboutie | 24h | SOC | Trimestriel |

**Runbook fuite PII** (P1) :
1. T+0 : Détection SIEM → ticket P1 SOC
2. T+15min : Confirmation analyste, isolation système (kill session, revoke token)
3. T+30min : Notification RSSI + DPO + Direction Juridique + DG
4. T+1h : Cellule de crise constituée
5. T+4h : Évaluation périmètre fuite (combien de personnes, quelles données)
6. T+24h : Décision notification CNIL (oui si risque élevé personnes concernées)
7. T+72h : Notification CNIL si requise (art. 33 RGPD), notification personnes concernées (art. 34 RGPD)
8. T+7j : Rapport RETEX, PAC corrections
9. T+30j : Validation correctifs déployés

---

<a name="section-b"></a>
## SECTION B — CONFORMITÉ RGPD

### B.1 DPIA (Analyse d'Impact Protection des Données)

**Méthodologie** : CNIL PIA + ENISA Guidelines + ISO/IEC 29134.

#### B.1.1 Caractérisation du traitement

| Élément | Description |
|---------|-------------|
| **Nom traitement** | Parapheur Digital — Workflow signature électronique |
| **Responsable de traitement** | [Société] — Direction Juridique |
| **DPO** | [Nom DPO] — dpo@[société].com |
| **Sous-traitants (art. 28)** | Universign (signature QES), AWS (hébergement), Okta (IAM), Splunk (logs), Twilio (SMS MFA) |
| **Finalités** | (1) Workflow validation engagements ; (2) Signature électronique opposable ; (3) Archivage probant ; (4) Audit/conformité |
| **Base légale (art. 6)** | Exécution contrat (clients/fournisseurs/salariés) + Obligation légale (archivage fiscal, comptable, social) + Intérêt légitime (audit interne) |
| **Durée conservation** | 10 ans après fin contrat (Code Commerce art. L123-22) — purge auto |
| **Volumétrie** | ~12 000 dossiers/an × 4,2 personnes concernées moyennes = ~50 000 personnes/an |

#### B.1.2 Cartographie des données personnelles (47 attributs PII)

| Catégorie | Attribut | Type RGPD | Sensibilité | Source | Stockage | Rétention | Base légale |
|-----------|---------|-----------|-------------|--------|----------|-----------|-------------|
| Identification | nom, prénom | Donnée standard | Faible | SAP HR | PG + S3 docs | 10 ans | Contrat |
| Identification | email pro | Donnée standard | Faible | SAP HR | PG | 10 ans | Contrat |
| Identification | téléphone pro | Donnée standard | Faible | SAP HR | PG | 10 ans | Contrat |
| Identification | matricule salarié | Donnée standard | Faible | SAP HR | PG | 10 ans | Contrat |
| Professionnel | fonction, service | Donnée standard | Faible | SAP HR | PG | 10 ans | Contrat |
| Professionnel | salaire (cas RH) | Donnée sensible métier | Élevée | SAP HR | PG chiffré field-level | 10 ans | Obligation légale |
| Professionnel | RIB (cas RH) | Bancaire | Élevée | SAP HR | Tokenization Vault | Suppression à fin contrat | Obligation légale |
| Identification | n° pièce ID (signataires QES) | Donnée standard | Élevée | Universign KYC | Universign (sous-traitant) | 6 ans après dernière signature | Obligation légale eIDAS |
| Comportementale | adresse IP | Donnée standard | Faible | Logs | Splunk | 1 an chaud + 6 ans froid | Obligation légale (audit) |
| Comportementale | timestamps actions | Donnée standard | Faible | Logs | PG audit | 10 ans | Obligation légale |
| Comportementale | user agent | Donnée standard | Faible | Logs | Splunk | 1 an | Intérêt légitime |
| Biométrique | empreinte WebAuthn | **Sensible art. 9** | **Très élevée** | Okta MFA | Okta (jamais exfiltré côté serveur) | À fin contrat | Consentement |

**Total** : 47 attributs PII, dont :
- 4 sensibles (art. 9 RGPD : biométrie WebAuthn, salaire, RIB, pièce ID)
- 43 standards
- **Aucune donnée santé, religion, orientation, origine** : exclues du périmètre

#### B.1.3 Analyse des risques RGPD

**Risques pour les personnes concernées** :

| ID | Risque | Source | Vraisemblance | Gravité | Score | Mesure |
|----|--------|--------|---------------|---------|-------|--------|
| R-RGPD-01 | Accès illégitime aux salaires | Validateur N+2 sans besoin | Possible (3) | Important (3) | 9 | Field-level encryption + accès log + droit-de-besoin |
| R-RGPD-02 | Modification frauduleuse signature | Tampering doc | Limitée (2) | Maximum (4) | 8 | Hash SHA-512 + horodatage TSA + Object Lock |
| R-RGPD-03 | Disparition documents archivés | Suppression accidentelle/malveillante | Limitée (2) | Important (3) | 6 | Object Lock COMPLIANCE + cross-region replication |
| R-RGPD-04 | Conservation excessive | Défaut purge auto | Possible (3) | Important (3) | 9 | Job Cron purge auto + dashboard rétention |
| R-RGPD-05 | Fuite massive (compromission) | Incident sécu | Limitée (2) | Maximum (4) | 8 | Chiffrement at-rest + segmentation + DLP |
| R-RGPD-06 | Profilage non autorisé | Usage IA non documenté | Maximum (4) | Important (3) | 12 | Pas d'IA pour décision automatisée art. 22 RGPD (validations sont humaines) |
| R-RGPD-07 | Transfert hors UE | Sous-traitant US (AWS) | Maximum (4) | Important (3) | 12 | Region eu-west-3 (Paris) + DPA + clauses standards UE |

**Mesures de réduction** : 7 risques identifiés, tous traités par mesures techniques+organisationnelles, score résiduel max = 4 (acceptable selon politique CNIL).

#### B.1.4 Validation DPIA

| Étape | Acteur | Date cible | Statut |
|-------|--------|------------|--------|
| Rédaction DPIA v0 | Chef projet + DPO | T+2 sem | À faire |
| Consultation parties prenantes | DPO + RSSI + métier | T+3 sem | À faire |
| Validation Comité Sécurité | RSSI + DPO + DSI | T+4 sem | À faire |
| Avis CNIL si requis (DPIA résiduel élevé) | DPO | T+6 sem | Conditionnel |
| Inscription registre traitements | DPO | T+5 sem | À faire |
| Revue annuelle | DPO | T+12 mois | Récurrent |

### B.2 Registre des traitements (art. 30 RGPD)

**Format CNIL conforme**, 1 fiche traitement principale + 4 fiches sous-traitants.

#### B.2.1 Fiche traitement principal

```
ID Traitement       : TR-PARAPH-001
Nom                 : Parapheur Digital
Responsable         : [Société] — DJ
DPO                 : [Nom DPO]
Date création       : 2026-Q3
Dernière MAJ        : [Date]

FINALITÉS
1. Validation workflow engagements financiers/RH/contrats
2. Signature électronique (AdES + QES)
3. Archivage probant 10 ans
4. Audit conformité interne et externe

BASE LÉGALE
- Art. 6.1.b RGPD : exécution contrat (employeur, fournisseur, client)
- Art. 6.1.c RGPD : obligation légale (Code Commerce, Code Travail, eIDAS)
- Art. 6.1.f RGPD : intérêt légitime (audit interne, sécurité)

PERSONNES CONCERNÉES
- Salariés (~3000)
- Candidats (~500/an)
- Fournisseurs (~1200 personnes contact)
- Clients (~800 personnes contact)
- Tiers signataires (~200/an : avocats, mandataires, partenaires)

CATÉGORIES DE DONNÉES
[Cf. tableau B.1.2 — 47 attributs]

DESTINATAIRES
- Internes : utilisateurs habilités selon RBAC/ABAC (cf. Phase 1)
- Externes : Universign (sous-traitant signature), AC qualifiée
- Légaux : autorités sur réquisition (URSSAF, fisc, justice)

TRANSFERTS HORS UE
- Aucun (region eu-west-3 Paris pour AWS, Universign basée en France)

DURÉE CONSERVATION
- Documents signés : 10 ans après fin contrat / clôture exercice
- Logs : 1 an actif + 6 ans archivage
- Données utilisateurs : tant que compte actif + 3 ans

MESURES SÉCURITÉ
[Cf. Section A — 47 contrôles ISO 27001]
```

### B.3 Droits des personnes concernées (art. 15-22)

#### B.3.1 Procédures opérationnelles

| Droit | Article | Délai légal | Procédure | Outil |
|-------|---------|------------|-----------|-------|
| **Accès** | Art. 15 | 1 mois (extensible 2) | Formulaire web → ticket DPO → extraction BDD + S3 → ZIP chiffré envoyé personne | Salesforce DPO module |
| **Rectification** | Art. 16 | 1 mois | Formulaire → vérification identité → MAJ source SAP HR → propagation parapheur | SAP HR (source) |
| **Effacement** | Art. 17 | 1 mois | Évaluation conditions (hors obligation légale) → si OK : crypto-shredding KMS + suppression PG + annotation S3 (Object Lock empêche delete, mais hash anonymisé) | Custom workflow |
| **Limitation** | Art. 18 | 1 mois | Marquage compte "limité" → blocage modifications mais pas accès historique | Flag user table |
| **Portabilité** | Art. 20 | 1 mois | Export JSON structuré données fournies par personne | Custom export |
| **Opposition** | Art. 21 | 1 mois | Évaluation intérêt légitime vs droits → décision DPO + DJ | Manuel |
| **Décision automatisée** | Art. 22 | N/A | Aucune décision automatisée (validations humaines) | N/A |

**Cas particulier "droit à l'oubli"** sur documents signés sous Object Lock COMPLIANCE :
- Object Lock empêche suppression physique (légitime : conservation légale 10 ans)
- Stratégie : **crypto-shredding** = destruction de la clé KMS rendant document illisible
- Anonymisation des métadonnées BDD (nom → "PERSONNE_ANONYMISÉE_xxxx")
- Conservation hash + audit trail (pour preuve traçabilité)
- Documenté formellement avec personne (réponse DPO type)

#### B.3.2 SLA réponse aux demandes

| Indicateur | Cible | Mesure |
|-----------|-------|--------|
| Délai réponse moyen | <15 jours | Salesforce DPO |
| Délai max | 30 jours (extensible 60 si complexe) | Idem |
| Taux respect délai légal | 100% | Idem |
| Taux satisfaction personne | >85% | Survey post-réponse |
| Demandes par an estimées | 80-120 | Benchmark secteur |

### B.4 Privacy by Design / by Default (art. 25)

**Mesures intégrées dès la conception** :

| Mesure | Implémentation |
|--------|----------------|
| **Minimisation** | Seuls 47 attributs nécessaires collectés (vs 80 disponibles SAP HR) |
| **Pseudonymisation** | Logs : email → hash SHA-256 (audit forensique seulement déchiffrable par DPO+RSSI) |
| **Chiffrement par défaut** | TDE BDD + SSE-KMS S3 + TLS 1.3 |
| **Cloisonnement** | Field-level encryption salaire/RIB (clés différentes par filiale) |
| **Anonymisation à terme** | Crypto-shredding au-delà 10 ans (rétention max légale) |
| **Default = privé** | Documents en draft non visibles (sauf initiateur), brouillons purgés J+30 |
| **Logs minimaux** | Pas d'enregistrement actions de consultation simple (RGPD vs traçabilité = compromis art. 32) |

### B.5 Sous-traitants RGPD (art. 28)

| Sous-traitant | Rôle | Pays données | DPA signé | Audit dispo | Certifications |
|---------------|------|--------------|-----------|-------------|----------------|
| Universign | Signature QES | France | OUI v2024 | SOC 2 + eIDAS | PSCo qualifié ANSSI |
| AWS | Hébergement | France (eu-west-3) | OUI (DPA AWS) | SOC 2/3, ISO 27001/17/18, HDS | + certifications crypto |
| Okta | IAM | EU (Frankfurt) | OUI | SOC 2, ISO 27001 | FedRAMP Moderate |
| Splunk Cloud | SIEM logs | EU | OUI | SOC 2 | ISO 27001 |
| Twilio | SMS MFA fallback | US (Privacy Shield invalidé → SCC + suppl.) | OUI | SOC 2 | + Standard Contractual Clauses UE |
| HashiCorp HCP Vault | Secrets | EU | OUI | SOC 2 | ISO 27001 |

**Évaluation risque sous-traitants** : annuelle (questionnaire + audit pour critiques).

---

<a name="section-c"></a>
## SECTION C — CONFORMITÉ eIDAS & SIGNATURE ÉLECTRONIQUE

### C.1 Cadre réglementaire eIDAS (Règlement UE 910/2014)

#### C.1.1 Niveaux de signature

| Niveau | Garanties | Cas d'usage parapheur | Code Civil |
|--------|-----------|----------------------|------------|
| **SES** (Simple) | Faible — case à cocher, signature scannée | Bons de commande <5K€ | Art. 1366 (recevabilité) mais charge preuve attaquant |
| **AdES** (Avancée) | Identification signataire + lien unique + intégrité doc + certificat | Dossiers 5K€-50K€ | Art. 1366 + 1367 al. 1 |
| **QES** (Qualifiée) | AdES + certificat qualifié AC qualifiée + dispositif sécurisé (HSM) | Dossiers ≥50K€, contrats clients/fournisseurs stratégiques, RH cadres | Art. 1366 + 1367 al. 2 (présomption fiabilité = équivalent signature manuscrite) |

**Décision projet** :
- 78% volumes → AdES (suffisant juridiquement pour <50K€)
- 22% volumes → QES (engagements >50K€ ou stratégiques) — couvre 87% des montants
- 0% SES (rejeté car preuves faibles)

#### C.1.2 Stratégie PSCo (Prestataire de Services de Confiance qualifié)

**Choix Universign** (alternatives évaluées : DocuSign, ChamberSign, Yousign) :

| Critère | Universign | DocuSign EU | Yousign | Winner |
|---------|-----------|-------------|---------|--------|
| Statut PSCo qualifié ANSSI | OUI | OUI | OUI | Égalité |
| HSM EAL4+ | OUI (eHerkenning + Thales) | OUI | OUI | Égalité |
| Souveraineté FR | **OUI (siège Paris)** | NON (US) | OUI | **Universign / Yousign** |
| API documentation | Excellente | Excellente | Bonne | DocuSign / Universign |
| Coût/signature QES | 0,80€ | 1,20€ | 0,95€ | **Universign** |
| Coût/signature AdES | 0,15€ | 0,30€ | 0,20€ | **Universign** |
| Volume négocié 12K/an | -25% remise | -15% | -10% | **Universign** |
| SLA dispo | 99,9% | 99,95% | 99,5% | DocuSign |
| Référence secteur public FR | OUI (DGFiP, CDC) | Limitée | OUI | **Universign / Yousign** |
| Support local FR | OUI (Paris) | Anglais | OUI | **Universign / Yousign** |

**Décision** : **Universign** retenu (souveraineté + coût + références secteur public client).

### C.2 Architecture signature électronique

#### C.2.1 Flux de signature QES

```
┌──────────────────────────────────────────────────────────────────────┐
│ 1. PRÉPARATION                                                        │
│    User clic "Signer"                                                 │
│    ↓                                                                  │
│    Backend parapheur prépare PDF + métadonnées                        │
│    ↓                                                                  │
│    Hash SHA-512 calculé                                               │
│    ↓                                                                  │
│    Appel API Universign /v3/transactions/signature                    │
│      ↓ POST {document_hash, signer_id, level: "qualified"}            │
└──────────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────┐
│ 2. AUTHENTIFICATION FORTE (eIDAS QES requirement)                     │
│    Universign vérifie identité signataire (1ère fois)                 │
│      - Pièce identité scan + OCR                                      │
│      - Selfie video (liveness check)                                  │
│      - Vérification adresse (via SAP HR data)                         │
│      - Signature certificat qualifié émis (validité 1 an)             │
│    ↓                                                                  │
│    Auth signataire (signatures suivantes) :                           │
│      - SMS OTP + selfie OU                                            │
│      - WebAuthn (FIDO2) avec certificat qualifié                      │
└──────────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────┐
│ 3. SIGNATURE                                                          │
│    User consentement explicite ("Je signe en pleine connaissance")    │
│    ↓                                                                  │
│    Hash document → HSM Universign                                     │
│    ↓                                                                  │
│    HSM signe ECDSA-P384 (clé privée NEVER quitte HSM)                 │
│    ↓                                                                  │
│    Signature retournée au backend parapheur                           │
└──────────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────┐
│ 4. HORODATAGE TSA (RFC 3161)                                          │
│    Signature → TSA Universign (Time Stamping Authority qualifiée)     │
│    ↓                                                                  │
│    TSA retourne horodatage signé (preuve temps universel)             │
└──────────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────┐
│ 5. ENRICHISSEMENT PAdES-LTA (Long Term Archive)                       │
│    Embed dans PDF :                                                   │
│      - Signature CMS                                                  │
│      - Certificat signataire + chaîne complète                        │
│      - CRL/OCSP statuts révocation au moment T                        │
│      - Horodatage TSA                                                 │
│    ↓ Format conforme ETSI EN 319 142 (PAdES Baseline B-LTA)           │
└──────────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────┐
│ 6. ARCHIVAGE                                                          │
│    PDF signé → S3 Object Lock COMPLIANCE 10 ans                       │
│    Métadonnées + hash → BDD audit (immutable)                         │
│    Notification signataire + initiateur                               │
└──────────────────────────────────────────────────────────────────────┘
```

#### C.2.2 Vérification long terme

**Problème** : certificats signataires expirent (1 an), AC peut disparaître, algorithmes deviennent obsolètes.

**Solution PAdES-LTA + Augmentation** :
- Tous les 8 ans : **augmentation horodatage** = ajout nouvel horodatage TSA scellant signature + horodatages précédents
- Conservation indéfinie même si infrastructure originale disparaît
- Validation possible 30+ ans après signature

**Procédure validation forensique** :
```
Input : document.pdf (signé)
1. Extract signature CMS du PDF
2. Extract certificat signataire
3. Vérifier chaîne certif (signataire → AC qualifiée → racine ANSSI)
4. Vérifier non-révocation au temps T (CRL/OCSP archivés dans PAdES-LTA)
5. Vérifier horodatage TSA (chaîne TSA → racine TSA qualifiée ANSSI)
6. Si augmenté : vérifier chaîne horodatages successifs
7. Recalculer hash document
8. Vérifier hash = hash signé
9. Output : VALID / INVALID + raison
```

### C.3 Conformité technique eIDAS

#### C.3.1 Checklist conformité QES (eIDAS art. 26 + RTS)

| Exigence | Article | Implémentation | Validation |
|----------|---------|----------------|------------|
| Lien unique signataire | Art. 26.a | Cert qualifié unique par user | Audit Universign |
| Identification signataire | Art. 26.b | KYC initial fort (pièce + selfie + adresse) | Conformité PSCo |
| Sous contrôle exclusif signataire | Art. 26.c | MFA + consentement explicite par signature | UX flow |
| Détection altération doc | Art. 26.d | Hash SHA-512 + signature CAdES | Test technique |
| Certificat qualifié | Art. 28 | AC qualifiée Universign (Trusted List ANSSI) | Liste de confiance UE |
| Dispositif qualifié (QSCD) | Art. 29 | HSM EAL4+ Universign certifié | Certif CC EAL4+ |
| Horodatage qualifié | Art. 42 | TSA qualifiée Universign | Trusted List |
| Preservation long terme | Art. 34 | PAdES-LTA + augmentation 8 ans | Audit annuel |

**Coût conformité** :
- Audit eIDAS annuel par CAB : 12 K€/an
- Conservation Trusted List : inclus PSCo
- Tests interopérabilité (DSS) : 8 K€ setup

#### C.3.2 Interopérabilité

**Validation par tiers** : tout document signé QES doit être vérifiable par n'importe quel outil conforme eIDAS (Adobe Reader, EU DSS, etc.)

**Tests d'interopérabilité** :
- Signature → vérification Adobe Acrobat Pro : OK attendu
- Signature → vérification EU DSS Demo (https://ec.europa.eu/digital-building-blocks/DSS/webapp-demo) : OK attendu
- Signature → vérification ChamberSign Validator : OK attendu

### C.4 Aspects juridiques signature électronique

#### C.4.1 Force probante (droit français)

| Niveau signature | Code Civil | Force probante | Renversement charge preuve |
|------------------|-----------|----------------|---------------------------|
| Manuscrite | Art. 1366 | Référence | Sur attaquant |
| QES eIDAS | Art. 1367 al.2 | **Équivalent manuscrite + présomption fiabilité** | Sur attaquant (preuve compromission ou KO infra) |
| AdES | Art. 1367 al.1 | Recevable | Sur signataire (doit prouver fiabilité) |
| SES | Art. 1366 | Recevable mais faible | Sur signataire (très lourd) |

**Implication projet** : pour engagements >50K€, **QES indispensable** car en cas contentieux, l'attaquant doit prouver que la signature est fausse (et non l'inverse).

#### C.4.2 Cas particuliers (formalisme renforcé)

Certains actes nécessitent formalisme spécifique :

| Acte | Exigence | Solution |
|------|---------|----------|
| Caution personne physique (>1500€) | Mention manuscrite (Art. L341-2 Code Conso) | Saisie texte signataire + horodatage = équivalent jurisprudence Cass. 13/03/2024 |
| Mandat immobilier | Acte notarié | Hors scope parapheur — externalisation notaire |
| Bail commercial >12 ans | Acte authentique | Idem |
| CDI cadre dirigeant | Recommandé écrit | QES + envoi LRE (Lettre Recommandée Électronique) qualifiée |
| Procuration bancaire | Selon banque | QES généralement acceptée si banque eIDAS-compatible |

#### C.4.3 Cas international

| Pays/zone | Reconnaissance QES eIDAS | Action |
|-----------|--------------------------|--------|
| **UE 27** | OUI (règlement direct) | Aucune |
| **Royaume-Uni** | OUI (UK Electronic Signatures Regulation 2002, post-Brexit equivalence) | Aucune |
| **Suisse** | OUI (loi SCSE, équivalence eIDAS) | Aucune |
| **USA** | Reconnaissance contractuelle (UETA/E-Sign Act) mais pas équivalent QES | Clauses contrat OK + DocuSign si client US exige |
| **Chine** | NON | Signature manuscrite + scan + tampon entreprise |
| **Russie** | NON | Signature manuscrite |

---

<a name="section-d"></a>
## SECTION D — PLAN DE TESTS & STRATÉGIE QA

### D.1 Stratégie de test (pyramide ISTQB)

```
                    ┌────────────┐
                    │   E2E      │  10% (47 scénarios)
                    │  (Cypress) │
                ┌───┴────────────┴───┐
                │   Integration       │  20% (180 tests)
                │   (REST Assured)    │
            ┌───┴─────────────────────┴───┐
            │   Component / Service       │  25% (250 tests)
            │   (Spring Boot Test)        │
        ┌───┴─────────────────────────────┴───┐
        │   Unit                              │  45% (1200 tests)
        │   (JUnit 5 + Mockito + Vitest)      │
        └─────────────────────────────────────┘
```

**Couverture cible** :
- Unit tests : 80% lignes / 75% branches (SonarQube quality gate)
- Integration : 100% endpoints REST critiques
- E2E : 100% golden paths + 40% scenarios alternatifs
- Pas de release prod si quality gate KO

### D.2 Catalogue de tests par type

#### D.2.1 Tests fonctionnels

**Périmètre** : 47 user stories Phase 1 + 87 règles métier Phase 2 = 134 cas à tester.

**Outils** :
- Backend : JUnit 5, AssertJ, Mockito, Testcontainers (PostgreSQL + Redis + Kafka in-container)
- Frontend : Vitest, React Testing Library, MSW (Mock Service Worker)
- E2E : Cypress 13 + Cucumber BDD

**Exemple test BDD** (US-D3-001 Signature QES) :
```gherkin
Feature: Signature QES dossier >50K€

  Scenario: Validateur signe contrat 75K€ avec QES Universign
    Given Marie est validateur N+2 authentifiée avec MFA WebAuthn
    And un dossier "Contrat-2026-0042" de 75 000€ est en attente signature
    When elle clique "Signer en QES"
    And confirme via Universign avec son certificat qualifié
    Then la signature PAdES-LTA est apposée sur le document
    And le dossier passe au statut "SIGNED"
    And un horodatage TSA est inclus
    And le document est archivé S3 Object Lock 10 ans
    And l'audit log enregistre l'action avec hash SHA-512
    And l'initiateur reçoit notification email
    And le dashboard KPI signatures-mois s'incrémente
```

#### D.2.2 Tests non-fonctionnels

##### D.2.2.1 Performance (objectifs SLA)

| Indicateur | Cible | Outil | Charge nominale | Charge pic |
|-----------|-------|-------|----------------|-----------|
| Latence API P95 | <500ms | Gatling + Datadog | 50 req/s | 200 req/s |
| Latence API P99 | <1500ms | Gatling | 50 req/s | 200 req/s |
| Latence signature QES P95 | <8s (incluant Universign) | Gatling | 5/min | 20/min |
| Throughput dossiers/jour | 80 nominal / 240 pic | Gatling | 12K/an = ~50/j | 240 burst fin mois |
| Disponibilité service | 99,9% (8,76h indispo/an) | Datadog | 24/7 | 24/7 |
| RTO (recovery time objective) | 4h | DR drill trimestriel | - | - |
| RPO (recovery point objective) | 15 min (replication continue) | - | - | - |

##### D.2.2.2 Tests de charge (Gatling scenarios)

| Scenario | Description | Profil charge | Critère succès |
|----------|-------------|---------------|----------------|
| LT-01 Nominal | Workflow standard 8h-18h | Ramp 0→50 req/s sur 1h, plateau 1h | P95 <500ms, 0 erreur |
| LT-02 Pic fin de mois | Burst clôture mensuelle | Ramp 0→200 req/s en 5 min, plateau 30 min | P95 <800ms, <0,1% erreurs |
| LT-03 Stress | Recherche limite | Ramp 0→500 req/s, plateau jusqu'à dégradation | Identifier point rupture |
| LT-04 Soak | Endurance 48h | Plateau 30 req/s sur 48h | Pas de fuite mémoire/connexions |
| LT-05 Spike | Pic brutal | 0→500 req/s en 10s | Auto-scaling déclenche, récupération <2 min |

##### D.2.2.3 Tests sécurité

| Type | Outil | Fréquence | Périmètre |
|------|-------|-----------|-----------|
| **SAST** (Static) | Snyk Code + SonarQube | Chaque commit (CI) | 100% code |
| **DAST** (Dynamic) | OWASP ZAP automated | Hebdomadaire | URLs publiques + API |
| **Dependency scan** | Snyk Open Source + Dependabot | Quotidien | package.json, pom.xml |
| **Container scan** | Trivy + Snyk Container | Build CI | Tous Dockerfiles |
| **IaC scan** | Checkov + tfsec | Chaque PR Terraform | Modules Terraform |
| **Secrets scan** | GitLeaks + TruffleHog | Pré-commit + CI | Repo entier + history |
| **Pentest externe** | Cabinet certifié PASSI | Annuel + avant chaque release majeure | Tout le périmètre |
| **Bug Bounty** | HackerOne (privé) | Continu | Production scope |
| **Audit RGPD** | DPO + cabinet externe | Annuel | Conformité RGPD |
| **Audit eIDAS** | CAB qualifié | Annuel | Conformité PSCo |
| **Audit ISO 27001** | Lloyd's Register / BSI | Tous les 3 ans (avec surveillance annuelle) | SMSI |
| **Red Team** | Optionnel (post-MVP) | Bi-annuel | Adversary emulation MITRE |

##### D.2.2.4 Tests accessibilité (WCAG 2.1 AA)

| Outil | Type | Couverture |
|-------|------|-----------|
| axe-core (CI) | Automatisé | 80% problèmes courants |
| Lighthouse | Audit | Score >90 cible |
| NVDA + JAWS | Manuel screen reader | 23 écrans top |
| Keyboard navigation | Manuel | 100% fonctions |
| Test utilisateurs handicap | Panel 8 personnes | Tests UX critiques |

##### D.2.2.5 Tests compatibilité

| Catégorie | Cible | Validation |
|-----------|------|------------|
| **Browsers** | Chrome 120+, Edge 120+, Firefox 120+, Safari 17+ | BrowserStack |
| **Mobile** | iOS 16+, Android 12+ | Appium + appareils réels |
| **Résolutions** | 1280×720 → 2560×1440, mobile 360×640 | Responsive Cypress viewports |
| **Performance bas débit** | 3G slow (1.5 Mbps) | Lighthouse throttling |

#### D.2.3 Tests métier (exécutés par utilisateurs clés)

**UAT (User Acceptance Testing)** : 4 semaines, 25 utilisateurs représentatifs (multi-filiales, multi-rôles).

| Phase UAT | Durée | Participants | Scope |
|-----------|-------|--------------|-------|
| Sprint UAT 1 | 1 sem | 10 initiateurs | Création dossiers types courants |
| Sprint UAT 2 | 1 sem | 8 validateurs | Workflows validation séquentiel/parallèle |
| Sprint UAT 3 | 1 sem | 5 signataires | Signatures AdES/QES |
| Sprint UAT 4 | 1 sem | 25 mixed | Bout-en-bout réels (dossiers production parallel run) |

**Critères acceptation UAT** :
- 0 anomalie bloquante
- ≤5 anomalies majeures (corrigées avant Go Live)
- ≥85% satisfaction (score CSAT)
- ≥80% taux complétion tâches sans aide

### D.3 Environnements de test

| Env | Usage | Données | Hébergement |
|-----|-------|--------|-------------|
| **DEV** | Dev quotidien | Dataset fixe anonymisé | AWS dev account |
| **TEST** | Tests intégration auto | Dataset fixe | AWS dev account |
| **STAGING** | Tests métier UAT + perf | Données prod anonymisées (snapshot mensuel) | AWS staging account (iso prod) |
| **PROD** | Production | Réelles | AWS prod account |
| **DR** | Disaster Recovery | Réplication prod | AWS prod (other region) |

**Anonymisation données prod → staging** :
- Noms/emails : remplacés par fakers cohérents
- Salaires : ±20% randomisation
- RIB : tokens aléatoires
- Documents PDF : header standard + corps anonymisé
- Outil : Tonic.ai ou solution custom

### D.4 Pipeline CI/CD avec gates qualité

```
┌─────────────────────────────────────────────────────────────────────┐
│  COMMIT → GitLab CI                                                  │
└─────────────────────────────────────────────────────────────────────┘
   ↓
┌─────────────────────────────────────────────────────────────────────┐
│  STAGE 1 : BUILD & UNIT TESTS (≤5 min)                               │
│  - Compile Java + TS                                                 │
│  - JUnit + Vitest                                                    │
│  - Quality gate : couverture ≥80%, 0 test failure                    │
└─────────────────────────────────────────────────────────────────────┘
   ↓ (si vert)
┌─────────────────────────────────────────────────────────────────────┐
│  STAGE 2 : SECURITY & QUALITY (≤8 min)                               │
│  - SonarQube : 0 bug critique, ≤5 vulns mineures                     │
│  - Snyk SAST : 0 vuln critique/haute                                 │
│  - Snyk Deps : 0 CVE critique                                        │
│  - GitLeaks : 0 secret                                               │
│  - License scan : compatibilité OSS                                  │
└─────────────────────────────────────────────────────────────────────┘
   ↓
┌─────────────────────────────────────────────────────────────────────┐
│  STAGE 3 : INTEGRATION TESTS (≤15 min)                               │
│  - Testcontainers PG/Redis/Kafka                                     │
│  - REST Assured 100% endpoints                                       │
│  - Schema contracts (Spring Cloud Contract)                          │
└─────────────────────────────────────────────────────────────────────┘
   ↓
┌─────────────────────────────────────────────────────────────────────┐
│  STAGE 4 : DOCKER BUILD & SCAN (≤5 min)                              │
│  - Multi-stage Dockerfile                                            │
│  - Trivy scan : 0 vuln critique                                      │
│  - Cosign sign image                                                 │
└─────────────────────────────────────────────────────────────────────┘
   ↓
┌─────────────────────────────────────────────────────────────────────┐
│  STAGE 5 : DEPLOY DEV → AUTO TEST E2E (≤20 min)                      │
│  - Helm deploy AWS EKS                                               │
│  - Smoke tests Cypress                                               │
│  - DAST OWASP ZAP                                                    │
└─────────────────────────────────────────────────────────────────────┘
   ↓ (manual approval)
┌─────────────────────────────────────────────────────────────────────┐
│  STAGE 6 : DEPLOY STAGING → UAT                                      │
│  - Blue/green deployment                                             │
│  - Tests perf Gatling                                                │
│  - UAT métier                                                        │
└─────────────────────────────────────────────────────────────────────┘
   ↓ (validation COPIL)
┌─────────────────────────────────────────────────────────────────────┐
│  STAGE 7 : DEPLOY PROD                                               │
│  - Canary 5% → monitoring 1h → 100% si OK                            │
│  - Datadog dashboard surveillance                                    │
│  - Rollback auto si error rate >0,5%                                 │
└─────────────────────────────────────────────────────────────────────┘
```

**Quality Gates obligatoires (bloquants)** :
- Couverture <80% → bloqué
- Vuln critique Snyk → bloqué
- Test E2E failure → bloqué
- Performance regression P95 >+20% → bloqué
- Quality gate SonarQube KO → bloqué

### D.5 Plan de tests global (planning)

| Sprint | Tests préparés | Tests exécutés | Bug fixing |
|--------|---------------|----------------|------------|
| S1-2 | Cadrage stratégie + setup outils | - | - |
| S3-4 | Test plans US (134 cas) | Tests unit dev en parallèle | - |
| S5-6 | Automatisation E2E Cypress | Test campagne intégration | Bugs critiques only |
| S7-8 | Préparation UAT + données anonymisées | UAT Sprint 1 (initiateurs) | Cycle 1 |
| S9-10 | Pentest externe lancé | UAT Sprint 2 (validateurs) + perf | Cycle 2 |
| S11-12 | Audit eIDAS + RGPD | UAT Sprint 3 (signataires) | Cycle 3 |
| S13-14 | Pre-prod | UAT Sprint 4 (e2e parallel run) + tests régression | Final |

---

<a name="section-e"></a>
## SECTION E — REGISTRE DES RISQUES SÉCURITÉ & CONFORMITÉ (P×I)

| ID | Risque | Catégorie | P (1-5) | I (1-5) | P×I | Stratégie | Plan d'action | Owner |
|----|--------|-----------|---------|---------|-----|-----------|---------------|-------|
| RS-01 | Compromission clé QES (HSM) | Sécurité crypto | 1 | 5 | 5 | Atténuation | HSM EAL4+ + accès physique restreint + monitoring 24/7 | RSSI |
| RS-02 | Fuite massive PII via SQL injection | Sécurité applicative | 2 | 5 | 10 | Atténuation | SAST + ORM + WAF + revue code obligatoire | RSSI/Dev |
| RS-03 | Sanction CNIL non-conformité RGPD | Conformité | 2 | 5 | 10 | Atténuation | DPIA + registre + droits + DPO + audit annuel | DPO |
| RS-04 | Indisponibilité Universign >24h | Sécurité opérationnelle | 2 | 4 | 8 | Atténuation + transfert | SLA + fallback AdES + assurance Universign | DSI |
| RS-05 | Compromission compte admin (RCE) | Sécurité applicative | 2 | 5 | 10 | Atténuation | PAM + JIT + MFA WebAuthn + audit logs | RSSI |
| RS-06 | Ransomware infrastructure | Sécurité opérationnelle | 2 | 5 | 10 | Atténuation + transfert | Backup immutable + EDR + cyber insurance 25M€ | RSSI |
| RS-07 | Insider malveillant (DBA exfiltre) | Sécurité organisationnelle | 2 | 5 | 10 | Atténuation | UEBA SIEM + DLP + bastion + revue accès trimestrielle | RSSI |
| RS-08 | Phishing ciblé validateur (BEC) | Sécurité humaine | 4 | 4 | 16 | Atténuation | Formation phishing trimestrielle + DMARC + alerte links | RSSI |
| RS-09 | Vulnérabilité 0-day librairie tiers (Log4Shell-like) | Sécurité applicative | 3 | 5 | 15 | Atténuation | Snyk continu + Bug Bounty + patch <48h critique | Dev/RSSI |
| RS-10 | Conformité eIDAS perdue (Universign perd qualif) | Conformité | 1 | 4 | 4 | Acceptation/Transfert | Veille trusted list + plan B Yousign | DJ |
| RS-11 | Non-conformité SecNumCloud (perte client public) | Conformité commerciale | 2 | 4 | 8 | Atténuation | Hébergement OVH SecNumCloud + audit ANSSI | DSI |
| RS-12 | Demande RGPD non traitée délais | Conformité | 3 | 3 | 9 | Atténuation | Salesforce DPO module + alertes + reporting mensuel | DPO |
| RS-13 | Audit ISO 27001 KO (perte certif) | Conformité | 2 | 3 | 6 | Atténuation | SMSI documenté + revues internes trimestrielles | RSSI |
| RS-14 | Performance dégradée pic charge | Sécurité opérationnelle | 3 | 3 | 9 | Atténuation | Tests perf + auto-scaling + monitoring SLA | DSI |
| RS-15 | Délais audit pentest dépassés | Projet | 3 | 3 | 9 | Atténuation | Cabinet sélectionné T-2 mois + budget réservé | RSSI |
| RS-16 | Formation utilisateurs insuffisante (mauvais usage) | Adoption | 4 | 3 | 12 | Atténuation | Plan formation 4h obligatoire + certif interne + supports vidéo | RH/DSI |
| RS-17 | Litige juridique signature contestée | Conformité juridique | 1 | 5 | 5 | Atténuation | QES eIDAS + horodatage + journal Universign | DJ |
| RS-18 | Coûts récurrents Universign explosent (volume) | Financier | 3 | 3 | 9 | Atténuation | Contrat plafond annuel + benchmark concurrence biennal | DAF |
| RS-19 | Dépendance fournisseur unique (HSM Thales) | Souveraineté | 2 | 3 | 6 | Acceptation | Standards PKCS#11 = portabilité, plan B Utimaco | RSSI |
| RS-20 | Évolution réglementaire (eIDAS 2.0 / EUDI Wallet) | Conformité | 4 | 3 | 12 | Veille | Roadmap intégration EUDI Wallet 2027 + budget réservé | DJ/DSI |

**Risques critiques (P×I ≥ 12)** : RS-08, RS-09, RS-16, RS-20 → plans d'action prioritaires Sprint 1-3.

---

<a name="section-f"></a>
## SECTION F — PLAN D'IMPLÉMENTATION & BUDGET

### F.1 Planning détaillé 14 semaines

| Sem | Workstream Sécurité | Workstream RGPD | Workstream eIDAS | Workstream QA |
|-----|--------------------|-----------------|------------------|---------------|
| S1 | Threat modeling STRIDE final | DPIA v0 rédigé | Sélection PSCo confirmée | Stratégie tests + outils |
| S2 | Setup IAM Okta + groupes RBAC | Registre traitements v1 | Contrat Universign signé | Setup CI/CD pipeline |
| S3 | Architecture HSM + commande Thales | Procédures droits RGPD documentées | Setup KYC Universign | Suite tests unit (couverture >70%) |
| S4 | Vault + secrets management | Formation DPO + équipe | Tests intégration sandbox Universign | Tests intégration auto |
| S5 | WAF + Cloudflare config | DPIA validé COPIL | Premiers tests signature AdES end-to-end | E2E Cypress 50% |
| S6 | SIEM Splunk : 12 use cases critiques | Sous-traitants : DPA tous signés | Tests signature QES bout-en-bout | Tests accessibilité WCAG |
| S7 | Pentest interne (équipe rouge) | Audit interne RGPD | Validation horodatage TSA | UAT Sprint 1 (initiateurs) |
| S8 | Correctifs pentest interne | - | Tests interopérabilité Adobe + EU DSS | UAT Sprint 2 (validateurs) |
| S9 | Pentest externe (cabinet PASSI) | - | Audit eIDAS blanc (CAB) | UAT Sprint 3 (signataires) + Pentest |
| S10 | Correctifs pentest externe | Audit DPO externe | Correctifs audit eIDAS | Tests perf Gatling complets |
| S11 | Bug Bounty privé lance (HackerOne) | - | Validation finale eIDAS | UAT Sprint 4 (parallel run) |
| S12 | Audit ISO 27001 (surveillance) | Audit RGPD final | - | Tests régression complets |
| S13 | Pre-prod hardening | Validation finale DPO | - | Tests acceptation finale |
| S14 | Go Live monitoring renforcé | - | - | RETEX + plan amélioration |

### F.2 Budget détaillé

#### F.2.1 CAPEX Sécurité (235 K€)

| Poste | Détail | Coût |
|-------|--------|------|
| HSM Thales Luna 7 (2 unités HA) | Hardware + licences | 145 K€ |
| Setup HashiCorp Vault HCP Enterprise | Setup + 1 an | 35 K€ |
| Cloudflare WAF Enterprise | 1 an | 18 K€ |
| Splunk ES + use cases | Setup + 1 an | 28 K€ |
| Outils sécurité dev (Snyk, SonarQube licenses) | 1 an | 9 K€ |

#### F.2.2 CAPEX RGPD (95 K€)

| Poste | Détail | Coût |
|-------|--------|------|
| DPIA + registre + procédures | Cabinet juridique externe | 25 K€ |
| Formation DPO + équipe | 6 personnes × 5j | 18 K€ |
| Outil gestion droits RGPD (Salesforce DPO) | Setup + 1 an | 22 K€ |
| Audit RGPD initial externe | Cabinet | 15 K€ |
| Tonic.ai anonymisation données | Licence + setup | 15 K€ |

#### F.2.3 CAPEX eIDAS (65 K€)

| Poste | Détail | Coût |
|-------|--------|------|
| Setup Universign (intégration + KYC) | Universign | 22 K€ |
| Audit eIDAS blanc (CAB) | Cabinet | 12 K€ |
| Audit eIDAS officiel pré-prod | CAB qualifié | 14 K€ |
| Conseil juridique eIDAS spécialisé | 30 j cabinet | 17 K€ |

#### F.2.4 CAPEX QA (85 K€)

| Poste | Détail | Coût |
|-------|--------|------|
| Pentest externe initial | Cabinet PASSI | 25 K€ |
| Bug Bounty setup + 1er trimestre | HackerOne | 12 K€ |
| Outils tests perf (Gatling Enterprise) | 1 an | 8 K€ |
| BrowserStack | 1 an | 5 K€ |
| UAT (logistique, indemnités) | 25 users × 4 sem | 35 K€ |

**TOTAL CAPEX** : **480 K€**

#### F.2.5 OPEX récurrent annuel (145 K€)

| Poste | Coût annuel |
|-------|-------------|
| HSM Thales maintenance | 28 K€ |
| Universign signatures (12K AdES + 2K QES estimés) | 38 K€ |
| Cloudflare WAF | 18 K€ |
| Splunk ES + ingestion | 24 K€ |
| Snyk + SonarQube licenses | 9 K€ |
| Pentest annuel | 18 K€ |
| Bug Bounty | 8 K€ |
| Audit RGPD annuel | 8 K€ |
| Audit eIDAS annuel | 12 K€ |
| Audit ISO 27001 surveillance | 6 K€ |
| **TOTAL OPEX** | **169 K€/an** (vs 145 K€ initial — réajusté) |

### F.3 Critères GO/NO-GO Production

#### F.3.1 Critères GO obligatoires (tous bloquants)

**Sécurité** :
- ☐ Pentest externe : 0 vulnérabilité critique, ≤2 hautes corrigées
- ☐ ISO 27001 : 41 contrôles déployés et auditables
- ☐ HSM EAL4+ opérationnel + clés QES générées
- ☐ SIEM 12 use cases critiques actifs
- ☐ Tests sécurité CI 100% verts

**RGPD** :
- ☐ DPIA validée par DPO et COPIL
- ☐ Registre traitements à jour
- ☐ DPA signés avec 6 sous-traitants
- ☐ Procédures droits RGPD documentées et testées (1 demande blanc)
- ☐ DPO formé et opérationnel

**eIDAS** :
- ☐ Universign opérationnel + tests bout-en-bout 100 signatures réussies
- ☐ Audit eIDAS officiel : conformité QES validée
- ☐ Validation interopérabilité Adobe + EU DSS OK
- ☐ Procédure vérification long terme documentée

**QA** :
- ☐ Couverture tests >80% (unit + integration)
- ☐ UAT : 0 anomalie bloquante, ≤5 majeures corrigées
- ☐ Tests performance : SLA respectés
- ☐ Tests accessibilité : WCAG 2.1 AA
- ☐ Plan de réponse incident testé (1 drill exécuté)
- ☐ Plan DR testé (RTO 4h validé)

#### F.3.2 Critères WARNING (acceptables avec plan d'action)

- Audit ISO 27001 surveillance : non-conformités mineures avec plan correctif
- Bug Bounty : vulnérabilités basses tolérées si plan
- Performance pic charge : dégradation acceptable si auto-scaling fonctionne
- UAT : satisfaction 75-85% acceptable si plan UX continu

#### F.3.3 Critères NO-GO

- 1 vulnérabilité critique pentest non corrigée
- DPIA non validée
- Audit eIDAS KO
- ≥1 anomalie bloquante UAT
- SLA performance non atteints en prod
- Indisponibilité testée DR >4h

---

<a name="annexes"></a>
## ANNEXES

### Annexe 1 — Templates documents

- A1.1 Template DPIA (CNIL PIA software)
- A1.2 Template DPA sous-traitants
- A1.3 Template registre traitements (art. 30)
- A1.4 Template procédure droit accès
- A1.5 Runbook incident sécurité P1
- A1.6 Politique mots de passe
- A1.7 Politique rotation clés
- A1.8 Procédure validation signature long terme

### Annexe 2 — Référentiels & standards

| Référentiel | Version | Périmètre |
|-------------|---------|-----------|
| ISO/IEC 27001:2022 | 2022 | SMSI |
| ISO/IEC 27002:2022 | 2022 | Contrôles |
| ISO/IEC 27005:2022 | 2022 | Risques |
| ISO/IEC 27018:2019 | 2019 | Cloud + PII |
| ISO/IEC 29134:2017 | 2017 | DPIA |
| RGS v2.0 | 2014 | Crypto ANSSI |
| eIDAS Règlement 910/2014 | 2014 | Signature électronique |
| eIDAS RTS (UE 2015/1502, 2015/1505) | 2015 | Niveaux assurance + formats |
| OWASP ASVS | 4.0.3 | Sécu applicative |
| OWASP API Security Top 10 | 2023 | API |
| NIST SP 800-63B | r3 | Authentification |
| NIST SP 800-61r2 | r2 | Réponse incident |
| NIST SP 800-88 | r1 | Sanitization |
| MITRE ATT&CK | v15 | Modélisation menaces |
| CIS Benchmarks | v8 | Hardening |
| RGPD | UE 2016/679 | Protection données |
| LIL | FR 78-17 modifiée | Protection données FR |
| Code Civil | Art. 1366-1369 | Preuves électroniques |
| ETSI EN 319 142 | 1.1.1 | PAdES |
| ETSI EN 319 122 | 1.1.1 | CAdES |
| RFC 3161 | 2001 | Time Stamping |
| RFC 5280 | 2008 | X.509 PKI |

### Annexe 3 — Checklist conformité globale

**Sécurité (47 contrôles ISO 27001:2022)** : checklist détaillée par contrôle (cf. A.2)

**RGPD (12 obligations clés)** :
- [ ] Base légale documentée par finalité
- [ ] Information personnes (art. 13/14)
- [ ] Recueil consentement si applicable (art. 7)
- [ ] DPIA pour traitements à risque (art. 35)
- [ ] Registre traitements (art. 30)
- [ ] Privacy by Design + Default (art. 25)
- [ ] Droits personnes opérationnels (art. 15-22)
- [ ] DPO désigné (art. 37-39)
- [ ] DPA sous-traitants (art. 28)
- [ ] Sécurité (art. 32)
- [ ] Notification violations (art. 33-34)
- [ ] Transferts hors UE encadrés (chap. V)

**eIDAS (8 exigences QES)** : cf. C.3.1

### Annexe 4 — Glossaire

| Terme | Définition |
|-------|-----------|
| **AdES** | Advanced Electronic Signature — signature électronique avancée eIDAS |
| **AC** | Autorité de Certification |
| **ABAC** | Attribute-Based Access Control |
| **ANSSI** | Agence Nationale Sécurité Systèmes Information |
| **ASVS** | Application Security Verification Standard (OWASP) |
| **BEC** | Business Email Compromise |
| **CAB** | Conformity Assessment Body |
| **CC** | Common Criteria (certification sécurité) |
| **CNIL** | Commission Nationale Informatique Libertés |
| **CSPRNG** | Cryptographically Secure Pseudo-Random Number Generator |
| **DAST** | Dynamic Application Security Testing |
| **DLP** | Data Loss Prevention |
| **DPA** | Data Processing Agreement |
| **DPIA** | Data Protection Impact Assessment |
| **DPO** | Data Protection Officer |
| **EAL** | Evaluation Assurance Level (Common Criteria) |
| **eIDAS** | Electronic IDentification, Authentication, trust Services |
| **EUDI** | European Digital Identity |
| **FIDO2** | Fast IDentity Online (passkeys) |
| **HSM** | Hardware Security Module |
| **HDS** | Hébergeur Données de Santé |
| **IDOR** | Insecure Direct Object Reference |
| **JIT** | Just-In-Time access |
| **JWT** | JSON Web Token |
| **KMS** | Key Management Service |
| **KYC** | Know Your Customer |
| **MFA** | Multi-Factor Authentication |
| **MITRE ATT&CK** | Adversarial Tactics, Techniques & Common Knowledge |
| **OCSP** | Online Certificate Status Protocol |
| **PAM** | Privileged Access Management |
| **PAdES** | PDF Advanced Electronic Signature |
| **PASSI** | Prestataire Audit Sécurité SI (qualifié ANSSI) |
| **PII** | Personally Identifiable Information |
| **PSCo** | Prestataire de Services de Confiance qualifié |
| **QES** | Qualified Electronic Signature |
| **QSCD** | Qualified Signature Creation Device |
| **RBAC** | Role-Based Access Control |
| **RGS** | Référentiel Général de Sécurité |
| **RPO** | Recovery Point Objective |
| **RTO** | Recovery Time Objective |
| **SAST** | Static Application Security Testing |
| **SCC** | Standard Contractual Clauses |
| **SecNumCloud** | Référentiel sécurité cloud ANSSI |
| **SIEM** | Security Information and Event Management |
| **SOC** | Security Operations Center |
| **SSDLC** | Secure Software Development Life Cycle |
| **SSO** | Single Sign-On |
| **STRIDE** | Spoofing, Tampering, Repudiation, Info Disclosure, DoS, Elevation |
| **TDE** | Transparent Data Encryption |
| **TSA** | Time Stamping Authority |
| **UEBA** | User and Entity Behavior Analytics |
| **WAF** | Web Application Firewall |
| **WCAG** | Web Content Accessibility Guidelines |
| **WORM** | Write Once Read Many |

---

## CONCLUSION & PROCHAINES ÉTAPES

### Points clés à retenir
1. **480 K€ CAPEX + 169 K€/an OPEX** pour conformité Sécurité+RGPD+eIDAS niveau 4/5
2. **47 contrôles ISO 27001 + 12 obligations RGPD + 8 exigences eIDAS QES** déployés en 14 semaines
3. **4 décisions COPIL** à arbitrer (D1 niveau signature, D2 souveraineté, D3 crypto, D4 tests)
4. **20 risques** identifiés avec plan d'action, dont 4 critiques (P×I≥12)
5. **GO/NO-GO** clairement défini : 23 critères bloquants + critères warning

### Demande au COPIL
- **Validation budget 480 K€** + 169 K€/an récurrent
- **Arbitrage 4 décisions D1-D4** (impact stratégique souveraineté + juridique)
- **Mandat équipe** : RSSI, DPO, DJ, DSI co-pilotage
- **Comité sécurité hebdomadaire** pendant 14 semaines

### Documents associés
- Phase 1 : Spécifications détaillées (US, archi, UX) — `PHASE1-specifications-detaillees.md`
- Phase 2 : Spécifications métier (règles, cas usage, données) — `PHASE2-specifications-metier.md`
- Phase 3 : Conformité & Sécurité (présent document)
- Phase 4 : Architecture & Intégration (à produire) — application détaillée, intégration ERP, plan migration

---

**Validation requise** :

| Rôle | Nom | Date | Signature |
|------|-----|------|-----------|
| Chef de Programme | | | |
| RSSI | | | |
| DPO | | | |
| Direction Juridique | | | |
| DSI | | | |
| DAF | | | |
| Directeur Général | | | |

**Fin du document Phase 3 — v1.0**
