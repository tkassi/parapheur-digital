# Phase 4 v2 — Architecture & intégration

**Programme Parapheur Digital — Banque commerciale universelle marocaine cotée à la Bourse de Casablanca, classée OIV**

| Métadonnée | Valeur |
|---|---|
| Version | R2.0 — Avril 2026 |
| Auteur | Direction de programme |
| Statut | Pour validation COPIL Direction (DSI, RSSI, Architecte SI banque) |
| Documents amendés | Phase 4 v1 (8 microservices, AWS Paris, Java/Spring) |
| Confidentialité | Interne / Direction — diffusion restreinte |
| Pages | ~85 |

---

## Note de cadrage

Cette Phase 4 v2 remplace la Phase 4 v1 pour le contexte d'une **banque marocaine cotée OIV opérant en mode solo + Claude**.

**Ce qui change vs v1** :

| Domaine | v1 (UE / équipe) | v2 (Maroc / solo) |
|---|---|---|
| Microservices | 8 (Dossier, Workflow, Notification, Audit, Signature, Archivage, Bus, Reporting) | **5** (Dossier, Workflow, Notification, Audit, Adapters) — Signature et Archivage délégués aux plateformes banque |
| Backend | Java 21 + Spring Boot 3.2 | **Node.js 20 + Fastify + TypeScript** |
| Frontend | React 18 + TS + Material UI | **React 18 + TS + Vite + TailwindCSS + shadcn/ui** |
| BPMN | Camunda 8 + DMN | **State machine custom (XState + table d'états PG)** |
| Bus | Kafka 3.7 + Kafka Connect | **Redis 7 BullMQ + PG LISTEN/NOTIFY** |
| Search | Elasticsearch 8 | **PostgreSQL 16 full-text + GIN/JSONB** |
| Auth | Okta + WebAuthn FIDO2 | **AD banque + MFA OTP** existant |
| HSM | Thales Luna 7 EAL4+ | **Supprimé** — délégué plateforme e-sign banque |
| Stockage objets | S3 + Glacier + KMS | **MinIO ou stockage interne** (court terme) + **GED banque** (long terme) |
| Cloud | AWS Paris eu-west-3 + DR Frankfurt | **Cloud interne banque** (préféré) ou **DC Maroc Tier III+** — souveraineté OIV |
| Conteneurs | EKS Kubernetes | **Kubernetes interne banque** (si dispo) sinon **Docker + Compose / Swarm** |
| IaC | Terraform + Helm + ArgoCD | **Terraform + Helm + GitHub Actions** (pas ArgoCD — surdimensionné solo) |
| Observabilité | Prom + Graf + Loki + Tempo + Sentry | **Prom + Graf + Loki + Sentry self-host** (Tempo retiré) |
| Stratégie release | Blue/Green Canary 5/25/50/100 % | **Rolling update simple** + feature flags |
| DR | Active/Passive cross-region multi-AZ | **Sauvegarde quotidienne + restore < 24h cross-DC Maroc** |
| Migration | 5 K dossiers actifs uniquement | **Pas de migration** — greenfield + double saisie 3 mois |

**Périmètre fonctionnel inchangé** : ~12 K dossiers/an, signature interne uniquement, intégration AD + plateformes e-sign et GED internes.

---

## Sommaire

1. Vue d'architecture C4 (4 niveaux)
2. Cinq microservices détaillés
3. Stack technique
4. Modèle de données
5. Intégrations API et adapters
6. Sécurité de l'architecture
7. Hébergement et infrastructure
8. Stratégie de déploiement
9. Observabilité
10. Plan de continuité (PRA/PCA)
11. Pipeline CI/CD
12. Sizing et coûts
13. Risques architecture
14. Plan d'exécution P5-P10
15. Annexes

---

## 1. Vue d'architecture C4 (4 niveaux)

### 1.1 Niveau 1 — Système (contexte)

```
                       ┌─────────────────┐
                       │  COLLABORATEURS │
                       │     BANQUE      │
                       │ (web + mobile)  │
                       └────────┬────────┘
                                │ HTTPS
                                ▼
                       ╔════════════════════════════╗
                       ║   PARAPHEUR DIGITAL        ║
                       ║   (système central)        ║
                       ║                            ║
                       ║   - workflow signature     ║
                       ║   - validation engagements ║
                       ║   - traçabilité audit      ║
                       ╚═══╤══╤══╤══╤══╤══╤═════════╝
                           │  │  │  │  │  │
        ┌──────────────────┘  │  │  │  │  └──────────────────┐
        ▼                     ▼  ▼  ▼  ▼                     ▼
┌──────────────┐  ┌────────────┐  ┌─────────────┐  ┌──────────────┐
│ Plateforme   │  │ AD / LDAP  │  │  Plateforme │  │ Core banking │
│ E-SIGNATURE  │  │   banque   │  │     GED     │  │  (référentiel│
│ INTERNE      │  │            │  │   INTERNE   │  │   clients)   │
└──────────────┘  └────────────┘  └─────────────┘  └──────────────┘
        │                                                      │
        │              ┌──────────────┐  ┌─────────────┐       │
        └──────────────┤ SMTP banque  │  │ SMS Gateway │───────┘
                       └──────────────┘  └─────────────┘
                       ┌──────────────┐  ┌─────────────┐
                       │ SIEM / SOC   │  │  Annuaire   │
                       │   banque     │  │  délégations│
                       └──────────────┘  └─────────────┘
```

**Acteurs externes** :

| Acteur | Type | Interaction | Protocole |
|---|---|---|---|
| Collaborateurs banque | Utilisateurs internes | Web + PWA mobile | HTTPS |
| AD/LDAP banque | Système AuthN/AuthZ | Authentification + groupes | LDAPS |
| Plateforme e-signature interne | Service banque | Signature électronique | HTTPS + mTLS + REST |
| Plateforme GED interne | Service banque | Archivage long terme | HTTPS + mTLS + REST |
| Core banking | Référentiel clients | Lookup données client (rare — usage interne) | HTTPS + REST |
| SMTP banque | Service email | Notifications email | SMTPS |
| SMS Gateway | Service SMS | Notifications SMS | HTTPS + REST |
| SIEM / SOC banque | Sécurité | Logs centralisés + alertes | Syslog / Kafka / API banque |
| Annuaire délégations | Référentiel RH | Délégations vacances | HTTPS + REST |

### 1.2 Niveau 2 — Containers (vue logique)

```
┌─────────────────────────────────────────────────────────────────┐
│                       SYSTÈME PARAPHEUR                          │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Frontend Web (React 18 + Vite + TS + Tailwind)          │   │
│  │  - PWA mobile activée                                    │   │
│  │  - i18n FR/AR (RTL)                                      │   │
│  │  - Service Worker pour offline lecture                   │   │
│  └─────────────────────────────┬────────────────────────────┘   │
│                                │ HTTPS                            │
│                                ▼                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  API Gateway (Kong ou NGINX + Lua)                       │   │
│  │  - Auth JWT, Rate limiting, CORS, audit log              │   │
│  └──┬──────────┬──────────┬──────────┬──────────┬───────────┘   │
│     │          │          │          │          │                │
│     ▼          ▼          ▼          ▼          ▼                │
│  ┌──────┐  ┌────────┐  ┌────────┐  ┌──────┐  ┌──────────────┐   │
│  │ MS-1 │  │  MS-2  │  │  MS-3  │  │ MS-4 │  │   MS-5/6     │   │
│  │Dossier│ │Workflow│  │Notif.  │  │Audit │  │  Adapters    │   │
│  │      │  │        │  │        │  │      │  │  e-Sign + GED│   │
│  └──┬───┘  └───┬────┘  └───┬────┘  └──┬───┘  └──────┬───────┘   │
│     │          │           │           │             │           │
│     └──────────┼───────────┼───────────┼─────────────┘           │
│                ▼           │           │                          │
│           ┌──────────────────────────────────────────┐            │
│           │  PostgreSQL 16 (cluster primary/replica) │            │
│           │  - Schémas par MS                        │            │
│           │  - JSONB métadonnées                     │            │
│           │  - Full-text intégré                     │            │
│           │  - LISTEN/NOTIFY pour events             │            │
│           └──────────────────────────────────────────┘            │
│                            │                                      │
│           ┌──────────────────────────────────────────┐            │
│           │  Redis 7 (cache + queue BullMQ)          │            │
│           └──────────────────────────────────────────┘            │
│                            │                                      │
│           ┌──────────────────────────────────────────┐            │
│           │  MinIO ou stockage objet interne         │            │
│           │  (fichiers actifs — TTL 30 jours)        │            │
│           └──────────────────────────────────────────┘            │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼ Adapters MS-5/6 (mTLS + REST)
              ┌─────────────────────────────────────────┐
              │  Plateformes externes (banque)          │
              │  - E-signature interne                  │
              │  - GED interne                          │
              │  - AD/LDAP                              │
              │  - SMTP / SMS / Core banking            │
              └─────────────────────────────────────────┘
```

### 1.3 Niveau 3 — Composants (zoom MS-2 Workflow)

```
┌─────────────────────────────── MS-2 Workflow ───────────────────────────────┐
│                                                                              │
│  ┌──────────────┐  ┌─────────────┐  ┌──────────────┐  ┌─────────────────┐  │
│  │ HTTP Routes  │  │ Workflow    │  │ State        │  │  Rules Engine    │  │
│  │ (Fastify)    │→→│ Service     │→→│ Machine      │←→│  (DMN-like JSON) │  │
│  └──────────────┘  └─────────────┘  │ (XState)     │  └─────────────────┘  │
│                                     └──────────────┘                        │
│                                            │                                 │
│  ┌──────────────┐  ┌─────────────┐         ▼                                 │
│  │ Event        │  │ Notification│  ┌──────────────┐  ┌─────────────────┐  │
│  │ Publisher    │←─│ Dispatcher  │←─│ Transition   │→→│  Persistence    │  │
│  │ (PG NOTIFY + │  │             │  │ Handler      │  │  (Prisma + PG)  │  │
│  │  BullMQ)     │  └─────────────┘  └──────────────┘  └─────────────────┘  │
│  └──────────────┘                                                            │
│         │                                                                    │
│         ▼                                                                    │
│   Vers MS-3 Notification, MS-4 Audit, MS-5 e-Sign, MS-6 GED                 │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 1.4 Niveau 4 — Code (extrait — moteur d'état)

Voir Annexe A — extrait XState pour la machine d'état du workflow de validation engagement.

---

## 2. Cinq microservices détaillés

### 2.1 MS-1 Dossier

**Responsabilité** : CRUD dossier, métadonnées, recherche, gestion documents (avant signature).

| Endpoint | Méthode | Description |
|---|---|---|
| `/api/v1/dossiers` | GET | Liste paginée filtrée (RBAC + ABAC) |
| `/api/v1/dossiers` | POST | Création dossier |
| `/api/v1/dossiers/{id}` | GET | Détail dossier |
| `/api/v1/dossiers/{id}` | PATCH | Modification (avant soumission) |
| `/api/v1/dossiers/{id}/documents` | POST | Upload document (multipart, max 50 Mo) |
| `/api/v1/dossiers/{id}/documents/{docId}` | GET | Téléchargement document |
| `/api/v1/dossiers/{id}/submit` | POST | Soumission au workflow (transfère à MS-2) |
| `/api/v1/dossiers/search` | GET | Recherche full-text PostgreSQL |

**Données** : table `dossier` (UUID, créateur, type, montant MAD, métadonnées JSONB, statut, dates), table `dossier_document` (UUID, dossierId, nom, hash SHA-256, mime, taille, storageRef MinIO).

**Effort dev** : 25 JH.

### 2.2 MS-2 Workflow

**Responsabilité** : machine à états, application des règles métier (DMN-like), orchestration des transitions, déclenchement des actions (signature, notification, archivage).

**Modèle d'état (XState)** :

```typescript
const dossierMachine = createMachine({
  id: 'dossier',
  initial: 'draft',
  context: {
    dossierId: null,
    montant: 0,
    devise: 'MAD',
    typeDecision: null,
    signataires: [],
    historique: []
  },
  states: {
    draft: {
      on: { SUBMIT: 'pending_validation' }
    },
    pending_validation: {
      entry: 'evaluateRules',  // détermine le prochain valideur via règles DMN
      on: {
        VALIDATE: [
          { target: 'pending_signature', cond: 'isFinalValidator' },
          { target: 'pending_validation', cond: 'hasNextValidator', actions: 'assignNext' }
        ],
        REJECT: 'rejected',
        DELEGATE: { actions: 'delegateToUser' }
      }
    },
    pending_signature: {
      invoke: { src: 'callESignAdapter', onDone: 'signed', onError: 'sign_error' },
      on: {
        SIGN_CALLBACK: { target: 'signed', cond: 'isSignedSuccessfully' },
        SIGN_REJECTED: 'rejected'
      }
    },
    signed: {
      invoke: { src: 'callGEDAdapter', onDone: 'archived' }
    },
    archived: { type: 'final' },
    rejected: { type: 'final' },
    sign_error: {
      after: { 60000: { target: 'pending_signature', actions: 'retryWithBackoff' } }
    }
  }
});
```

**Règles métier (DMN-like)** : tables JSON paramétrables, 87 règles totales (cf. Phase 2 v2).

**Effort dev** : 40 JH (le plus complexe).

### 2.3 MS-3 Notification

**Responsabilité** : envoi email + SMS + push web, templating, tracking des envois, retry.

**Architecture** :

```
Event "notification.send" (BullMQ queue)
       │
       ▼
NotificationService
       │
       ├──→ TemplateRenderer (Handlebars) — FR + AR
       │
       ├──→ EmailDispatcher → SMTP banque (nodemailer)
       ├──→ SmsDispatcher → SMS Gateway (axios)
       └──→ PushDispatcher → Web Push API (futur)
       │
       ▼
  Log MS-4 + tracking table
```

**Templates** : 18 templates (création, validation requise, validation faite, rejet, signature requise, signature faite, archivage, rappel, escalade, etc.) × 2 langues (FR + AR) = 36 templates.

**Effort dev** : 12 JH.

### 2.4 MS-4 Audit

**Responsabilité** : journal immuable des événements, recherche, export, intégrité.

**Modèle** :

```sql
CREATE TABLE audit_log (
  id BIGSERIAL PRIMARY KEY,
  event_id UUID UNIQUE NOT NULL,
  occurred_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  actor_user_id VARCHAR(50) NOT NULL,
  actor_ip INET,
  actor_user_agent TEXT,
  event_type VARCHAR(80) NOT NULL,
  resource_type VARCHAR(50) NOT NULL,
  resource_id VARCHAR(100) NOT NULL,
  before JSONB,
  after JSONB,
  metadata JSONB,
  hash_chain CHAR(64) NOT NULL,  -- SHA-256 de (event + previous hash)
  signature TEXT
);
CREATE INDEX idx_audit_resource ON audit_log (resource_type, resource_id);
CREATE INDEX idx_audit_actor ON audit_log (actor_user_id, occurred_at);
CREATE INDEX idx_audit_event_type ON audit_log (event_type, occurred_at);
```

**Caractéristiques** :
- Append-only (révocation INSERT pour utilisateurs applicatifs ; seul rôle `audit_writer` autorisé)
- Hash chaîné par lot horaire (job batch calcule digest_hour)
- Pas de DELETE/UPDATE (revoke privileges)
- Conservation : 10 ans chaud + transfert GED post-clôture
- Export : API JSON paginée + CSV pour audit BAM/AMMC

**Effort dev** : 15 JH.

### 2.5 MS-5/6 Adapters e-Signature et GED

**Responsabilité** : façade API vers plateformes internes banque, gestion des erreurs, retry, circuit breaker, callback handling.

**Pattern Anti-Corruption Layer (ACL)** :

```typescript
// MS-5 e-Sign Adapter
class ESignAdapter {
  async requestSignature(req: SignRequest): Promise<SignRequestId> {
    const internalDto = this.mapToInternalDTO(req);  // adaptation au modèle plateforme banque
    return this.client.post('/api/v1/sign-requests', internalDto, {
      headers: { Authorization: this.getOAuth2Token() },
      mtlsCert: this.cert,
      retry: { attempts: 3, backoff: 'exponential' },
      circuitBreaker: 'esign-banque'
    });
  }
  
  async handleCallback(payload: any, signature: string): Promise<void> {
    if (!this.verifyHMAC(payload, signature)) throw new SecurityError();
    const event = this.mapFromInternalDTO(payload);
    await this.eventBus.publish('esign.callback', event);
  }
}
```

**Effort dev** : 18 JH (e-Sign) + 12 JH (GED) = 30 JH.

### 2.6 Synthèse effort microservices

| MS | Effort | Complexité |
|---|---|---|
| MS-1 Dossier | 25 JH | Moyenne |
| MS-2 Workflow | 40 JH | Élevée |
| MS-3 Notification | 12 JH | Faible |
| MS-4 Audit | 15 JH | Moyenne |
| MS-5 Adapter e-Sign | 18 JH | Moyenne (intégration) |
| MS-6 Adapter GED | 12 JH | Moyenne (intégration) |
| **Total backend** | **122 JH** | – |
| Frontend Web + PWA | 60 JH | Moyenne |
| Setup infra + IaC + CI/CD | 25 JH | – |
| Tests E2E + perf + sécurité | 25 JH | – |
| **Total dev** | **232 JH** | – |
| Conformité (cf. P3 v2) | 43 JH | – |
| Documentation + recette + mise en prod | 40 JH | – |
| **Total Build** | **315 JH** | – |

---

## 3. Stack technique

### 3.1 Vue d'ensemble

| Couche | Choix | Justification mode solo |
|---|---|---|
| **Frontend** | React 18 + TypeScript + Vite + TailwindCSS + shadcn/ui + TanStack Query | Stack mainstream, écosystème mature, génération rapide via Claude |
| **State front** | Zustand (état UI) + TanStack Query (cache serveur) | Léger, peu de boilerplate |
| **Forms** | React Hook Form + Zod | Validation typée alignée backend |
| **i18n** | i18next + react-i18next + RTL via Tailwind | FR + AR avec direction CSS |
| **Backend** | Node.js 20 LTS + Fastify 4 + TypeScript | Performance, sécurité, écosystème |
| **ORM** | Prisma 5 | Type-safe, migrations versionnées, intuitif |
| **Validation** | Zod | Schémas partagés front/back |
| **Auth** | Passport.js + ldap-strategy + JWT | Standard, bien documenté |
| **Workflow** | XState 5 + state machine custom + table d'états PG | Pas de Camunda — surdimensionné solo |
| **Queue** | BullMQ 5 (Redis) | Gestion jobs background, retry, scheduling |
| **Events** | PostgreSQL LISTEN/NOTIFY + BullMQ | Pas de Kafka — volumétrie 50/jour |
| **Cache** | Redis 7 | Multi-usage (sessions, cache, queue) |
| **DB** | PostgreSQL 16 | Robuste, JSONB, full-text, LISTEN/NOTIFY |
| **Search** | PostgreSQL `tsvector` + GIN | Pas d'Elasticsearch (overkill 12K/an) |
| **Stockage objets** | MinIO ou stockage interne banque | S3-compatible, on-prem |
| **API Gateway** | Kong OSS ou NGINX + Lua | Maturité, OSS, plugins |
| **Tests** | Vitest + Playwright + k6 + Supertest + Testcontainers | Stack JS unifiée |
| **Lint/Format** | ESLint + Prettier + Stylelint | Standard |
| **CI/CD** | GitHub Actions (ou GitLab CI banque) | Banque hébergée, gratuit/inclus |
| **IaC** | Terraform 1.7 + Helm 3 | Standard, multi-cloud |
| **Conteneurs** | Docker + Compose (dev) + Helm (prod) | Standard, simple |
| **Orchestration** | Kubernetes interne banque (si dispo) sinon Docker Swarm ou Compose en VMs | Adaptation contexte banque |
| **Observabilité** | Prometheus + Grafana + Loki + Sentry self-host | OSS, suffisant solo |
| **APM** | Sentry pour erreurs, OpenTelemetry pour traces (optionnel) | Stack légère |
| **Secrets** | HashiCorp Vault ou secret manager interne banque | Sécurité critique |
| **PKI** | PKI banque + Let's Encrypt si exposition publique | Cohérence banque |

### 3.2 Pourquoi pas Java/Spring (vs v1)

| Critère | Java/Spring (v1) | Node.js/TS (v2) |
|---|---|---|
| Productivité solo + Claude | Bon | **Excellent** (Claude génère TS très efficacement, écosystème npm) |
| Verbosité | Élevée | **Faible** |
| Cohérence stack | Front TS / Back Java (deux mondes) | **Full-stack TS** (types partagés via tRPC ou Zod) |
| Performance | Excellente | Très bonne (suffisante 50/jour) |
| Empreinte mémoire | ~512 Mo / pod | **~150 Mo / pod** |
| Démarrage à froid | Plusieurs secondes | **< 1 seconde** |
| Apprentissage | Connu mais lourd solo | Plus léger |
| Tests | JUnit + Mockito | Vitest (rapide, syntaxe moderne) |
| Maturité bancaire | Très élevée | Élevée (Goldman Sachs, JPM, Capital One l'utilisent) |

**Décision** : Node.js/TypeScript pour cohérence full-stack et productivité solo. Claude génère du TypeScript de qualité production avec très peu de friction.

### 3.3 Pourquoi pas Camunda BPM (vs v1)

| Critère | Camunda 8 (v1) | XState + custom (v2) |
|---|---|---|
| Modélisation BPMN | Visuelle (Modeler) | Code (machine declarative) |
| Apprentissage | Important (Camunda + DMN + Zeebe) | Faible (XState + JSON) |
| Empreinte | Cluster Zeebe + ElasticSearch + Operate + Tasklist (~4 services additionnels) | Bibliothèque incluse dans MS-2 |
| Coût licence | Camunda 8 EE payant pour features avancées | 0 |
| Volumétrie cible | Excellente pour millions/jour | Excellente pour milliers/jour |
| Versioning workflows | Natif (process versions) | Manuel via Prisma migrations |

**Décision** : XState + table d'états PG. Suffisant pour 50 workflows/jour et < 10 versions concurrentes.

### 3.4 Pourquoi pas Kafka (vs v1)

| Critère | Kafka 3.7 (v1) | BullMQ + PG NOTIFY (v2) |
|---|---|---|
| Volumétrie max | Millions/sec | Milliers/sec |
| Volumétrie cible | 50 events/jour ouvré | Identique |
| Empreinte | Cluster 3 brokers + ZK ou KRaft + Connect + Schema Registry | Redis (déjà nécessaire) + PG (déjà nécessaire) |
| Apprentissage | Important | Faible |
| Garanties | Exactly-once, replay, partitions | At-least-once, retry, scheduling |

**Décision** : BullMQ + PG LISTEN/NOTIFY. Aligne sur la volumétrie réelle.

---

## 4. Modèle de données

### 4.1 Schéma PostgreSQL (extrait — tables principales)

```sql
-- Schéma 'dossier'
CREATE SCHEMA IF NOT EXISTS dossier;

CREATE TABLE dossier.dossier (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  reference VARCHAR(30) UNIQUE NOT NULL,  -- DOS-2026-001234
  type_decision VARCHAR(50) NOT NULL,      -- engagement-credit | validation-rh | etc.
  createur_user_id VARCHAR(50) NOT NULL,
  createur_direction VARCHAR(20),
  createur_agence VARCHAR(20),
  montant NUMERIC(18, 2),
  devise CHAR(3) DEFAULT 'MAD',
  metadata JSONB DEFAULT '{}'::jsonb,
  statut VARCHAR(30) NOT NULL DEFAULT 'draft',
  workflow_id UUID,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  submitted_at TIMESTAMPTZ,
  closed_at TIMESTAMPTZ,
  ged_archive_id VARCHAR(100),  -- ID dans la GED après archivage
  CONSTRAINT chk_statut CHECK (statut IN (
    'draft','pending_validation','pending_signature','signed','archived','rejected','cancelled'
  ))
);
CREATE INDEX idx_dossier_createur ON dossier.dossier (createur_user_id, created_at DESC);
CREATE INDEX idx_dossier_statut ON dossier.dossier (statut, created_at DESC);
CREATE INDEX idx_dossier_metadata ON dossier.dossier USING GIN (metadata);
CREATE INDEX idx_dossier_search ON dossier.dossier
  USING GIN (to_tsvector('french', reference || ' ' || coalesce(metadata->>'objet', '')));

CREATE TABLE dossier.dossier_document (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  dossier_id UUID NOT NULL REFERENCES dossier.dossier(id) ON DELETE CASCADE,
  nom VARCHAR(255) NOT NULL,
  mime_type VARCHAR(100),
  taille_octets BIGINT,
  hash_sha256 CHAR(64) NOT NULL,
  storage_ref VARCHAR(500) NOT NULL,  -- minio://bucket/key
  upload_user_id VARCHAR(50) NOT NULL,
  uploaded_at TIMESTAMPTZ DEFAULT now()
);

-- Schéma 'workflow'
CREATE SCHEMA IF NOT EXISTS workflow;

CREATE TABLE workflow.workflow_instance (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  dossier_id UUID NOT NULL REFERENCES dossier.dossier(id),
  state_current VARCHAR(50) NOT NULL,
  state_context JSONB DEFAULT '{}'::jsonb,
  workflow_definition_id VARCHAR(50) NOT NULL,
  workflow_definition_version SMALLINT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  closed_at TIMESTAMPTZ
);

CREATE TABLE workflow.workflow_transition (
  id BIGSERIAL PRIMARY KEY,
  workflow_id UUID NOT NULL REFERENCES workflow.workflow_instance(id),
  from_state VARCHAR(50),
  to_state VARCHAR(50) NOT NULL,
  event VARCHAR(50) NOT NULL,
  actor_user_id VARCHAR(50),
  occurred_at TIMESTAMPTZ DEFAULT now(),
  payload JSONB
);

CREATE TABLE workflow.workflow_task (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  workflow_id UUID NOT NULL REFERENCES workflow.workflow_instance(id),
  type VARCHAR(50) NOT NULL,  -- 'validation' | 'signature' | 'review'
  assignee_user_id VARCHAR(50),
  assignee_role VARCHAR(50),
  statut VARCHAR(30) NOT NULL DEFAULT 'pending',
  due_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  decision VARCHAR(20),  -- 'approved' | 'rejected' | 'delegated'
  comment TEXT,
  metadata JSONB
);
CREATE INDEX idx_task_assignee ON workflow.workflow_task (assignee_user_id, statut);

-- Schéma 'audit'
CREATE SCHEMA IF NOT EXISTS audit;

CREATE TABLE audit.audit_log (
  id BIGSERIAL PRIMARY KEY,
  event_id UUID UNIQUE NOT NULL DEFAULT gen_random_uuid(),
  occurred_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  actor_user_id VARCHAR(50) NOT NULL,
  actor_ip INET,
  actor_user_agent TEXT,
  event_type VARCHAR(80) NOT NULL,
  resource_type VARCHAR(50) NOT NULL,
  resource_id VARCHAR(100) NOT NULL,
  before JSONB,
  after JSONB,
  metadata JSONB,
  hash_chain CHAR(64) NOT NULL
);
CREATE INDEX idx_audit_resource ON audit.audit_log (resource_type, resource_id, occurred_at DESC);
CREATE INDEX idx_audit_actor ON audit.audit_log (actor_user_id, occurred_at DESC);
CREATE INDEX idx_audit_event ON audit.audit_log (event_type, occurred_at DESC);

REVOKE INSERT, UPDATE, DELETE ON audit.audit_log FROM PUBLIC;
GRANT INSERT ON audit.audit_log TO audit_writer;

-- Schéma 'esign'
CREATE SCHEMA IF NOT EXISTS esign;

CREATE TABLE esign.sign_request (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  dossier_id UUID NOT NULL,
  document_id UUID NOT NULL,
  document_hash CHAR(64) NOT NULL,
  signature_level VARCHAR(10) NOT NULL,  -- SES | SEA | SEQ
  external_request_id VARCHAR(100),       -- ID plateforme banque
  signers JSONB NOT NULL,
  callback_received_at TIMESTAMPTZ,
  signed_document_ref VARCHAR(500),
  certificate_thumbprint VARCHAR(128),
  timestamp_token TEXT,
  statut VARCHAR(20) NOT NULL DEFAULT 'pending',
  expires_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

### 4.2 Volumétrie estimée

| Table | Volume an 1 | Volume an 5 (cumulé) | Croissance |
|---|---|---|---|
| `dossier` | 12 K | 60 K | + 12 K/an |
| `dossier_document` | 30 K | 150 K | × 2,5 dossier |
| `workflow_instance` | 12 K | 60 K | = dossier |
| `workflow_transition` | 80 K | 400 K | × ~7 par workflow |
| `workflow_task` | 50 K | 250 K | × ~4 par workflow |
| `audit_log` | 200 K | 1 M | × ~16 par dossier |
| `sign_request` | 12 K | 60 K | = dossier |

**Total stockage 5 ans estimé** : ~10 Go DB + ~250 Go fichiers (avant archivage GED). Très modéré.

### 4.3 Stratégie d'archivage

- Documents originaux post-signature : **GED banque** (long terme, 10 ans, conformité Code commerce)
- Stockage MinIO local : **TTL 30 jours** post-archivage GED (cache pour relecture rapide)
- DB chaud : **2 ans** données complètes
- DB tiède : **partition mensuelle** par date, anciennes partitions exportées en lecture seule
- DB froid : **GED banque** ou bucket archive — extraction CSV/JSON par lot

---

## 5. Intégrations API et adapters

### 5.1 Cartographie des intégrations

| # | Système banque | Direction | Volume estimé | Criticité | Protocole |
|---|---|---|---|---|---|
| 1 | AD/LDAP | Auth + groupes (sortant) | 40 req/jour ouvré (login + sync) | Critique | LDAPS |
| 2 | Plateforme e-signature | Sortant + callback entrant | 50 req/jour ouvré | Critique | HTTPS + mTLS REST |
| 3 | Plateforme GED | Sortant | 50 req/jour ouvré | Critique | HTTPS + mTLS REST |
| 4 | SMTP banque | Sortant | 200 emails/jour ouvré | Importante | SMTPS |
| 5 | SMS Gateway | Sortant | 50 SMS/jour ouvré | Importante | HTTPS REST |
| 6 | Core banking (lookup client) | Sortant | < 10 req/jour | Faible (rare en interne) | HTTPS REST |
| 7 | Annuaire délégations | Sortant | 1 sync/jour | Modérée | HTTPS REST |
| 8 | SIEM banque | Sortant (logs) | continu | Importante | Syslog ou Kafka banque |

**Total** : 8 intégrations vs 13 en v1 (suppression Workday HR, Salesforce, eDoc externe, etc.).

### 5.2 Adapter e-Signature (MS-5)

#### 5.2.1 Spécification API d'appel

```yaml
openapi: 3.0.3
paths:
  /api/v1/sign-requests:
    post:
      summary: Créer une demande de signature
      security:
        - oauth2_client_credentials: []
        - mtls: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [documentName, documentHash, signatureLevel, signers, callbackUrl]
              properties:
                documentId: { type: string, format: uuid }
                documentName: { type: string, maxLength: 255 }
                documentHash: { type: string, pattern: '^[a-f0-9]{64}$' }
                documentBase64: { type: string, format: byte }  # OU documentUrl
                signatureLevel: { type: string, enum: [SES, SEA, SEQ] }
                signers:
                  type: array
                  items:
                    type: object
                    required: [userId, order]
                    properties:
                      userId: { type: string }
                      email: { type: string, format: email }
                      phone: { type: string }
                      order: { type: integer, minimum: 1 }
                      role: { type: string }
                callbackUrl: { type: string, format: uri }
                expiresAt: { type: string, format: date-time }
                metadata: { type: object }
      responses:
        '202':
          description: Demande acceptée
          content:
            application/json:
              schema:
                type: object
                properties:
                  signRequestId: { type: string, format: uuid }
                  status: { type: string, enum: [PENDING] }
                  signatureUrl: { type: string, format: uri }
                  expiresAt: { type: string, format: date-time }
```

#### 5.2.2 Spécification callback entrant

```yaml
  /api/v1/esign/callback:  # endpoint exposé par le parapheur
    post:
      summary: Callback de la plateforme e-sign banque
      security:
        - mtls: []
        - hmac_signature: []  # header X-Signature
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [signRequestId, status, signedAt]
              properties:
                signRequestId: { type: string, format: uuid }
                status: { type: string, enum: [SIGNED, REJECTED, EXPIRED, ERROR] }
                signatureLevel: { type: string }
                signedDocumentUrl: { type: string }
                signedDocumentHash: { type: string }
                signers:
                  type: array
                  items:
                    type: object
                    properties:
                      userId: { type: string }
                      signedAt: { type: string, format: date-time }
                      certificateThumbprint: { type: string }
                      ipAddress: { type: string }
                timestampToken: { type: string }  # RFC 3161 base64
                auditTrail: { type: string }
      responses:
        '200': { description: Callback traité }
        '401': { description: Signature HMAC invalide }
        '422': { description: Payload invalide }
```

#### 5.2.3 Pattern Anti-Corruption Layer

```typescript
// Adapter — isole le modèle banque du modèle interne parapheur
class ESignBankAdapter implements ESignPort {
  async request(req: ParapheurSignRequest): Promise<ParapheurSignRequestId> {
    // Mapping modèle interne → modèle plateforme banque
    const bankRequest = {
      documentName: req.documentName,
      documentHash: req.documentHash,
      documentBase64: await this.fetchAndEncode(req.documentRef),
      signatureLevel: this.mapLevel(req.level),
      signers: req.signers.map(s => ({
        userId: s.matriculeBanque,
        email: s.email,
        phone: s.phone,
        order: s.order,
        role: s.parapheureRole
      })),
      callbackUrl: `${this.config.baseUrl}/api/v1/esign/callback`,
      expiresAt: req.expiresAt.toISOString(),
      metadata: { dossierId: req.dossierId, ... }
    };
    
    const response = await this.httpClient.post('/api/v1/sign-requests', bankRequest, {
      timeout: 30_000,
      retry: { attempts: 3, backoffMs: [1000, 5000, 15000] }
    });
    
    return new ParapheurSignRequestId(response.signRequestId);
  }
  
  async handleCallback(payload: unknown, signature: string): Promise<void> {
    if (!this.verifyHMAC(payload, signature)) {
      throw new SignatureVerificationError();
    }
    const validated = ESignCallbackSchema.parse(payload);  // Zod
    const event = this.mapToInternalEvent(validated);
    await this.eventBus.publish('esign.callback.received', event);
  }
}
```

#### 5.2.4 Circuit breaker

```typescript
const breaker = new CircuitBreaker({
  name: 'esign-banque',
  failureThreshold: 5,        // 5 échecs consécutifs
  resetTimeout: 60_000,       // tentative reset après 60s
  halfOpenMaxAttempts: 1,
  onOpen: () => logger.error('ESign breaker OPEN — mode dégradé activé'),
  onClose: () => logger.info('ESign breaker CLOSED — service rétabli')
});
```

**Mode dégradé** : si le breaker est ouvert > 30 min, l'interface bascule en "signature physique" — le document est imprimé, signé manuscritement, scanné, et ré-injecté avec hash + photo de preuve.

### 5.3 Adapter GED (MS-6)

```yaml
paths:
  /api/v1/documents:
    post:
      summary: Archiver un document signé
      security: [{ oauth2: [], mtls: [] }]
      requestBody:
        content:
          application/json:
            schema:
              type: object
              required: [name, category, retentionPolicy, fileBase64, metadata]
              properties:
                name: { type: string }
                category: { type: string }  # ex: 'engagement-credit-interne'
                retentionPolicy: { type: string }  # ex: '10-years-commercial'
                metadata:
                  type: object
                  required: [dossierId, signaturesCertificates]
                fileBase64: { type: string, format: byte }
      responses:
        '201':
          content:
            application/json:
              schema:
                type: object
                properties:
                  documentId: { type: string }
                  url: { type: string }
                  archivedAt: { type: string, format: date-time }

  /api/v1/documents/{documentId}:
    get:
      summary: Récupérer un document archivé
      security: [{ oauth2: [], mtls: [] }]
      responses:
        '200':
          content:
            application/octet-stream: {}
```

**Politique de rétention** : `10-years-commercial` configuré côté GED banque selon Code commerce 15-95 art. 22. Legal hold actif par défaut pour 6 mois post-signature (sécurité litige).

**Effort dev MS-6** : 12 JH.

### 5.4 Intégration AD/LDAP

```typescript
// Authentification
import passport from 'passport';
import LdapStrategy from 'passport-ldapauth';

passport.use(new LdapStrategy({
  server: {
    url: 'ldaps://ad.banque.ma:636',
    bindDN: vault.get('ad/bind_dn'),
    bindCredentials: vault.get('ad/bind_password'),
    searchBase: 'OU=Users,DC=banque,DC=ma',
    searchFilter: '(sAMAccountName={{username}})',
    searchAttributes: ['sAMAccountName','displayName','mail','memberOf','department','title'],
    tlsOptions: { ca: [readFileSync('ca-banque.crt')] }
  }
}));

// Sync groupes (job quotidien)
async function syncADGroups() {
  const users = await ldapClient.search({
    base: 'OU=Users,DC=banque,DC=ma',
    filter: '(memberOf=CN=Parapheur*,OU=Groups,DC=banque,DC=ma)'
  });
  for (const user of users) {
    await db.user.upsert({
      where: { matricule: user.sAMAccountName },
      update: { groupes: user.memberOf, direction: user.department },
      create: { matricule: user.sAMAccountName, ... }
    });
  }
}
```

**MFA** : OTP délégué à AD banque (ADFS + Azure MFA, ou Active Directory Federation Services + RSA SecurID, selon stack banque).

### 5.5 Notifications email + SMS

```typescript
// Email
import nodemailer from 'nodemailer';
const transporter = nodemailer.createTransport({
  host: 'smtp.banque.ma',
  port: 587,
  secure: false,
  requireTLS: true,
  auth: { user: ..., pass: vault.get('smtp/password') }
});

// SMS — via gateway interne banque
async function sendSMS(phone: string, message: string) {
  return axios.post('https://sms.banque.ma/api/send', {
    to: phone,
    message: message,
    sender: 'BANQUE'
  }, { headers: { Authorization: `Bearer ${vault.get('sms/token')}` }});
}
```

---

## 6. Sécurité de l'architecture

(Cf. P3 v2 § 5 — détails complets. Synthèse ici.)

| Couche | Mesures clés |
|---|---|
| Périmètre | mTLS interne, TLS 1.3 externe, segmentation réseau (zones DMZ / app / data / external) |
| Identité | AD banque + MFA OTP + JWT court (8h) + refresh token + révocation Redis |
| Autorisation | RBAC × ABAC évaluée backend systématiquement |
| Données | Chiffrement at-rest AES-256-GCM (clés cloud banque KMS / Vault), chiffrement in-transit TLS 1.3 |
| Audit | Journal append-only hash-chaîné, NTP synchronisé, export AMMC sous 5j |
| Secrets | HashiCorp Vault ou Secret Manager banque, rotation trimestrielle |
| Code | SAST (Semgrep, SonarQube), DAST (OWASP ZAP), Snyk dépendances, Trivy conteneurs |
| Pipeline | Gates qualité bloquants (coverage 80 %, vuln critiques), revue automatique |
| Conformité | DNSSI 11 chap, ISO 27001:2022 (35 contrôles applicables), Loi 09-08, Loi 43-20, Loi 05-20 |

---

## 7. Hébergement et infrastructure

### 7.1 Options d'hébergement (rappel décision D-R1)

| Option | Avantages | Inconvénients | Coût indicatif | Recommandation |
|---|---|---|---|---|
| **A. Cloud interne banque** | Souveraineté, intégration native, gouvernance unifiée, coût marginal | Capacité à confirmer, SLA interne | Refacturé interne | **Préféré** si capacité disponible |
| **B. DC Maroc Tier III+** (OVH Casa, Maroc Datacenter, N+ONE, AWS BFM si dispo) | Souveraineté, conformité OIV, latence faible utilisateurs | Setup plus long, coût opex | ~6 000-9 000 MAD/mois | **Retenu** si A indisponible |
| **C. AWS Paris (eu-west-3)** | Maturité, scaling, écosystème | Hors Maroc → autorisation CNDP + dérogation OIV improbable | ~5 000 MAD/mois | **Déconseillé** OIV |
| **D. Azure / GCP** | Maturité | Souveraineté, dérogation OIV improbable | ~5 000 MAD/mois | **Déconseillé** OIV |

**Recommandation forte** : option A (cloud interne banque) ou option B (DC Maroc Tier III+).

### 7.2 Architecture infrastructure (option B — DC Maroc, illustrative)

```
                    Internet
                       │
                       ▼
                  WAF banque (DMZ)
                       │
                       ▼
                  Reverse Proxy (NGINX)
                       │
                       ▼
                  API Gateway (Kong)
                       │
        ┌──────────────┴──────────────┐
        │                             │
        ▼                             ▼
   Zone App (private subnet)     Zone Data (private subnet)
   ┌─────────────────────┐       ┌─────────────────────┐
   │ K8s ou Docker Swarm │       │ PostgreSQL primary  │
   │ Pods/services :     │       │ + 1 replica streaming│
   │ - Frontend          │       │                     │
   │ - MS-1 à MS-6       │       │ Redis cluster       │
   │                     │       │ (master + replica)  │
   │                     │       │                     │
   │                     │       │ MinIO 2 noeuds      │
   └─────────────────────┘       └─────────────────────┘
        │                             │
        └──────────────┬──────────────┘
                       ▼
              VPN / Direct Connect
                       │
                       ▼
              Réseau interne banque
              ├── AD/LDAP
              ├── Plateforme e-signature
              ├── Plateforme GED
              ├── SMTP, SMS Gateway
              └── SIEM
```

### 7.3 Sizing

| Composant | CPU | RAM | Stockage | Réseau |
|---|---|---|---|---|
| Frontend (NGINX statique) | 1 vCPU | 512 Mo | 5 Go | 100 Mbps |
| API Gateway Kong | 2 vCPU | 1 Go | 10 Go | 200 Mbps |
| MS-1 à MS-6 (6 services × 2 instances) | 12 × 0,5 vCPU | 12 × 256 Mo | 12 × 5 Go | – |
| PostgreSQL primary | 4 vCPU | 16 Go | 200 Go SSD | 200 Mbps |
| PostgreSQL replica | 4 vCPU | 16 Go | 200 Go SSD | – |
| Redis (master + replica) | 2 × 1 vCPU | 2 × 2 Go | 2 × 20 Go | – |
| MinIO (2 noeuds) | 2 × 2 vCPU | 2 × 4 Go | 2 × 500 Go | – |
| Observabilité (Prom + Graf + Loki) | 2 vCPU | 4 Go | 100 Go | – |
| **Total** | **~30 vCPU** | **~60 Go RAM** | **~1,5 To stockage** | – |

**Empreinte modeste** — un cluster de 4-5 VMs (8 vCPU, 16 Go RAM chacune) ou 1-2 nœuds Kubernetes physiques suffit.

### 7.4 Coûts cloud option B (DC Maroc)

| Poste | Mensuel (MAD) | Annuel (MAD) |
|---|---|---|
| 4 VMs (8 vCPU, 16 Go, 100 Go SSD) | 4 000 | 48 000 |
| Stockage objet (MinIO ou S3-compatible 500 Go) | 500 | 6 000 |
| Backup offsite (500 Go × 30 jours rétention) | 800 | 9 600 |
| Réseau (10 Mbps dédié + IP fixes) | 500 | 6 000 |
| Monitoring intégré | 200 | 2 400 |
| **Total** | **6 000** | **72 000** |

---

## 8. Stratégie de déploiement

### 8.1 Environnements

| Env | Usage | Données | Volumes |
|---|---|---|---|
| **dev** | Dev + tests locaux | Fictives | 1 utilisateur dev |
| **staging** | Tests intégration + UAT | Anonymisées (subset) | ~5-10 testeurs |
| **prod** | Production | Réelles | Tous utilisateurs banque |

### 8.2 Stratégie de release (rolling update simple)

```
git push → CI → Build image → Push registry → Helm upgrade
                                                    │
                                                    ▼
                                            Rolling update:
                                            - 1 pod à la fois
                                            - Readiness probes
                                            - Rollback auto si échec
```

**Pourquoi pas Blue/Green ou Canary** : volumétrie 50/jour ouvré = 1-2 utilisateurs simultanés en moyenne ; rolling update suffit. Feature flags (Unleash ou OpenFeature) gèrent les déploiements progressifs de fonctionnalités.

### 8.3 Cadence de release

| Type | Cadence | Procédure |
|---|---|---|
| Hotfix critique | À la demande | Hotfix branch + déploiement immédiat |
| Mineur (bug, ajustement) | Hebdomadaire (jeudi) | Tag + déploiement automatique |
| Majeur (feature) | Mensuel | Validation comité + communication users |

### 8.4 Feature flags

```typescript
import { Unleash } from 'unleash-client';

const ff = new Unleash({ url: '...', appName: 'parapheur', instanceId: '...' });

if (ff.isEnabled('seq_signature_for_high_value', { user })) {
  return signWithSEQ();
} else {
  return signWithSEA();
}
```

**Cas d'usage** : activation progressive d'une fonctionnalité (par direction, par type de dossier), kill-switch en cas d'incident.

---

## 9. Observabilité

### 9.1 Stack

| Outil | Usage | Hébergement |
|---|---|---|
| **Prometheus** | Métriques | Self-host |
| **Grafana** | Dashboards | Self-host |
| **Loki** | Logs centralisés | Self-host (ou SIEM banque) |
| **Sentry** (self-host) | Erreurs applicatives + alerting | Self-host |
| **OpenTelemetry** | Traces (optionnel) | Optionnel |
| **Alertmanager** | Routage alertes | Self-host |
| **PagerDuty / Opsgenie / SMS banque** | On-call | Selon banque |

### 9.2 Métriques clés (Golden Signals + métier)

| Métrique | Type | Seuil alerte |
|---|---|---|
| `http_request_duration_p95` (par endpoint) | Latence | > 500 ms |
| `http_request_duration_p99` | Latence | > 1 000 ms |
| `http_request_rate` | Trafic | – |
| `http_request_error_rate` (5xx) | Erreurs | > 1 % |
| `db_connection_pool_saturation` | Saturation | > 80 % |
| `redis_memory_used_bytes` | Saturation | > 80 % capacité |
| `bullmq_queue_size` | Saturation | > 100 jobs en attente |
| `bullmq_failed_jobs_rate` | Erreurs | > 5 % |
| `esign_callback_latency` | Métier | > 30 s |
| `esign_circuit_breaker_state` | Métier | open |
| `dossier_created_total` | Métier | – (suivi business) |
| `dossier_signed_total` | Métier | – |
| `dossier_signed_duration_avg` (cycle complet) | Métier | > 4h moyenne |

### 9.3 Logs structurés

```typescript
import pino from 'pino';
const logger = pino({
  level: process.env.LOG_LEVEL ?? 'info',
  redact: ['*.password', '*.token', '*.secret', '*.documentBase64'],  // masking
  formatters: {
    level: (label) => ({ level: label }),
    bindings: (b) => ({ pid: b.pid, hostname: b.hostname, service: 'ms-1-dossier' })
  },
  timestamp: pino.stdTimeFunctions.isoTime
});

logger.info({ userId, dossierId, action: 'create' }, 'Dossier créé');
```

**Format** : JSON structuré, ingestion Loki + indexation par `service`, `level`, `userId`, `dossierId`, `traceId`.

### 9.4 Alertes critiques

| Alerte | Condition | Sévérité | Destinataire |
|---|---|---|---|
| Service DOWN | health check 3 échecs consécutifs | S1 | Dev solo + RSSI |
| Erreur 5xx > 5 % | sur 5 minutes | S2 | Dev solo |
| Latence p95 > 1s | sur 10 minutes | S3 | Dev solo (heures ouvrées) |
| DB pool saturé | > 90 % sur 5 min | S2 | Dev solo |
| Disque > 90 % | – | S2 | Dev solo + DSI |
| Circuit breaker e-sign OPEN | > 5 minutes | S2 | Dev solo + DSI banque |
| Échec backup quotidien | – | S1 | Dev solo + DSI |
| Tentatives auth échouées > 50 sur 5 min (par IP) | – | S2 | RSSI + SOC |
| Audit log incohérence hash-chain | – | **S1** | RSSI + dev solo |

### 9.5 Tableaux de bord Grafana

| Dashboard | Audience | Contenu |
|---|---|---|
| **Vue technique** | Dev | Golden signals par service, infra, DB, Redis |
| **Vue métier** | DSI + métier | KPIs : dossiers/jour, cycle moyen, taux échec, top types |
| **Vue sécurité** | RSSI | Auth, 4xx/5xx, alertes sécurité, audit volume |
| **Vue conformité** | DPO + audit | Export logs MS-4, requêtes accès, demandes droits CNDP |

---

## 10. Plan de continuité (PRA / PCA)

(Cf. P3 v2 § 10 — détails complets. Synthèse ici.)

### 10.1 Cibles

| Métrique | Valeur | Source |
|---|---|---|
| RTO global | ≤ 4h | BAM 5/W/2017 |
| RPO global | ≤ 1h | BAM 5/W/2017 |
| Test annuel | Q4 | BAM + Loi 05-20 OIV |

### 10.2 Architecture PRA

- **Site primaire** : DC Maroc Casablanca (ou cloud banque DC1)
- **Site secondaire** : DC Maroc Rabat (ou cloud banque DC2) — cold standby
- **Sauvegardes** : quotidiennes chiffrées AES-256, rétention 30j + mensuel 12 mois + annuel 7 ans
- **PostgreSQL** : streaming replication async vers replica + WAL archive offsite
- **Redis** : reconstruction depuis PG (pas de RDB persistent critique)
- **Stockage objets** : réplication asynchrone primary → secondary
- **Application** : Helm chart redéployable < 30 min sur DC2

### 10.3 Procédure de bascule

| Étape | Délai cible |
|---|---|
| Détection incident DC1 | 5 min |
| Décision bascule (RSSI + DSI) | 30 min |
| Promotion replica PG → primary | 15 min |
| Redéploiement app sur DC2 | 30 min |
| DNS / load balancer pointage DC2 | 10 min |
| Tests fonctionnels | 30 min |
| Communication utilisateurs | en parallèle |
| **Total RTO** | **≤ 2h** (marge sur 4h cible) |

---

## 11. Pipeline CI/CD

### 11.1 Pipeline détaillé

```yaml
# .github/workflows/ci.yml (GitHub Actions)
name: CI/CD
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'pnpm' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm typecheck
      - run: pnpm test:unit --coverage
      - run: pnpm test:integration
      - uses: codecov/codecov-action@v4
        with: { fail_ci_if_error: true }
      - run: pnpm audit --audit-level=high
      - uses: snyk/actions/node@master
        with: { args: --severity-threshold=high }
        env: { SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }} }
      - uses: returntocorp/semgrep-action@v1
  
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t parapheur/ms-1:${{ github.sha }} .
      - uses: aquasecurity/trivy-action@master
        with:
          image-ref: parapheur/ms-1:${{ github.sha }}
          severity: CRITICAL,HIGH
          exit-code: '1'
      - run: docker push registry.banque.ma/parapheur/ms-1:${{ github.sha }}
  
  deploy-staging:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: helm upgrade --install parapheur ./helm -f values-staging.yaml --set image.tag=${{ github.sha }}
      - run: pnpm test:e2e -- --baseURL=https://staging.parapheur.banque.ma
  
  deploy-prod:
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    environment: production  # protection avec approbation manuelle
    runs-on: ubuntu-latest
    steps:
      - run: helm upgrade parapheur ./helm -f values-prod.yaml --set image.tag=${{ github.sha }}
      - run: pnpm test:smoke -- --baseURL=https://parapheur.banque.ma
```

### 11.2 Gates de qualité

| Gate | Outil | Bloquant ? |
|---|---|---|
| Lint | ESLint | Oui |
| Format | Prettier | Oui |
| TypeCheck | tsc | Oui |
| Tests unitaires + intégration | Vitest | Oui |
| Coverage ≥ 80 % | Codecov | Oui |
| SAST | Semgrep + SonarQube | Critiques bloquants |
| Dépendances | Snyk + npm audit | High+ bloquants |
| Conteneurs | Trivy | Critical+ bloquants |
| E2E staging | Playwright | Non bloquant (analyse manuelle) |
| Smoke test prod | Playwright | Bloquant + rollback auto |

---

## 12. Sizing et coûts

### 12.1 Effort développement (rappel)

| Lot | JH solo + Claude |
|---|---|
| P0 Kickoff | 15 |
| Setup IaC + CI/CD | 25 |
| Backend 5 microservices | 122 |
| Frontend Web + PWA | 60 |
| Intégrations API | 30 (inclus dans backend si on consolide) |
| Tests | 25 |
| Conformité | 43 (cf. P3) |
| Recette + doc + mise en prod | 28 |
| **Total Build** | **~315 JH** |

### 12.2 Coûts cash architecture & infra

| Poste | Mois | Annuel (MAD) |
|---|---|---|
| Cloud DC Maroc Tier III+ (4 VMs + stockage + backup + réseau) | 6 000 | 72 000 |
| Domaine + certificats SSL | – | 1 500 |
| Outillage dev (JetBrains, Figma, npm pro) | 700 | 8 000 |
| Claude Pro/API | 400 | 4 800 |
| Sentry self-host (infra incluse cloud) | – | 0 |
| Monitoring (Prometheus + Grafana + Loki — self-host) | – | 0 |
| **Total OPEX architecture** | – | **~86 300 MAD/an** |

(Hors conformité — cf. P3 v2 § 12 pour audit BAM, pentest, frais CNDP.)

---

## 13. Risques architecture

| ID | Risque | P | I | P×I | Mitigation |
|---|---|---|---|---|---|
| RA-01 | Plateforme e-sign banque indisponible / non agréée DGSSI | 3 | 4 | **12** | Audit P0 + plan B Barid eSign + mode dégradé signature physique |
| RA-02 | Plateforme GED banque indisponible | 2 | 4 | **8** | Stockage local 7j + retry batch + alerte DSI |
| RA-03 | Sous-dimensionnement (volumétrie sous-estimée) | 2 | 3 | **6** | Tests perf P9, scale horizontal possible (PG replica + 2 instances par MS) |
| RA-04 | Sur-dimensionnement (coût excessif) | 2 | 2 | **4** | Sizing révisé annuellement |
| RA-05 | Échec PRA test annuel | 2 | 4 | **8** | Test à blanc en P10, simulations trimestrielles |
| RA-06 | Dérive de version dépendances (vulnérabilités) | 4 | 3 | **12** | Snyk + Dependabot + revue trimestrielle |
| RA-07 | Migration PostgreSQL majeure (PG16 → PG17 dans 5 ans) | 3 | 2 | **6** | Procédure documentée, test sur staging, fenêtre maintenance |
| RA-08 | Perte expertise XState (workflow) | 2 | 4 | **8** | Doc exhaustive + machine déclarative + Claude restitue le contexte |
| RA-09 | Cloud banque indisponible pour le projet | 3 | 3 | **9** | Plan B DC Maroc Tier III+ (option B) |
| RA-10 | Indisponibilité AD banque | 1 | 5 | **5** | Cache JWT court, mode dégradé read-only |
| RA-11 | Compromission certificat mTLS | 1 | 5 | **5** | Rotation annuelle, révocation immédiate via PKI banque |
| RA-12 | Dérive secret JWT (clé privée RS256) | 1 | 5 | **5** | Rotation annuelle, multi-clés simultanées (current + previous), Vault |
| RA-13 | Bug critique en production | 3 | 4 | **12** | Tests ≥ 80 %, smoke tests prod, rollback automatique |
| RA-14 | Latence intégration core banking | 2 | 2 | **4** | Cache Redis, lookup async, timeout strict |
| RA-15 | Saturation BullMQ (queue backlog) | 2 | 3 | **6** | Workers scalables, alertes > 100 jobs |

**Top 5 risques architecture** : RA-01 (e-sign agrément), RA-06 (dépendances), RA-13 (bug prod), RA-09 (cloud), RA-02 (GED).

---

## 14. Plan d'exécution P5-P10

### 14.1 Phases

| Phase | Période | JH | Livrables clés |
|---|---|---|---|
| **P0** Kickoff | sem 1-4 | 15 | Audit API banque, pré-consultation CNDP, lettre de mission, ADR socle |
| **P5** Cadrage opérationnel | sem 5-8 | 25 | IaC Terraform, CI/CD pipeline, env dev/staging, OpenAPI socle, schéma DB initial |
| **P6** Build vague 1 (MVP) | sem 9-20 | 80 | MS-1 Dossier + MS-3 Notification + frontend MVP (création/lecture) + auth AD + adapters mocks |
| **P7** Build vague 2 | sem 21-34 | 90 | MS-2 Workflow réel + MS-5 e-Sign réel + MS-6 GED réel + i18n FR/AR + règles DMN MAD complètes |
| **P8** Build vague 3 | sem 35-42 | 50 | MS-4 Audit complet + intégrations finales (core banking, annuaire délégations) + reporting + tests perf + sécurité E2E |
| **P9** Recette & conformité | sem 43-48 | 30 | Audit cabinet inscrit BAM (5j), pentest cabinet agréé DGSSI (8j), dépôt CNDP, UAT, formation, plan remédiation |
| **P10** Mise en prod & hypercare | sem 49-54 | 25 | Go-live progressif (10 % → 50 % → 100 % users) + hypercare 4 sem + premier test PRA + transfert TMA Run |
| **Total Build** | **54 sem** | **315 JH** | – |
| **Run** | continu | 45 JH/an | TMA + évolutions + monitoring + audit annuel + pentest annuel |

### 14.2 Jalons

| Jalon | Date cible (T+sem) | Critère GO/NO-GO |
|---|---|---|
| **J0** | T+0 | Lettre de mission signée, accès banque, équipe sponsor identifiée |
| **J1** | T+4 | Audit API banque livré, ADR socle, environnements provisionnés |
| **J2** | T+20 | MVP démontrable (dossier + workflow simple + 1 type signature simulé) |
| **J3** | T+34 | Intégration complète e-sign + GED réelles, i18n FR/AR, règles métier MAD |
| **J4** | T+42 | Code complet, tests ≥ 80 %, audit externe BAM en cours |
| **J5** | T+48 | Audit OK, dépôt CNDP validé, UAT signée, plan remédiation pentest exécuté |
| **J6** | T+54 | Go-live + hypercare terminé + transfert TMA Run |

### 14.3 Points de contrôle

- **Hebdomadaire** : point sponsor banque (DSI ou DG) — 30 min, statut + risques + arbitrages
- **Bi-mensuel** : revue jalons, replanification si écart > 10 %
- **Trimestriel** : COPIL Direction — revue programme, décisions, budget
- **Mensuel** : reporting au comité sécurité banque

---

## 15. Annexes

### Annexe A — Extrait XState pour la machine d'état du dossier

```typescript
import { setup, assign } from 'xstate';

export const dossierMachine = setup({
  types: {
    context: {} as DossierContext,
    events: {} as DossierEvent
  },
  actions: {
    evaluateRules: assign({ /* moteur DMN */ }),
    assignNext: assign({ assignee: ({ context }) => nextValidator(context) }),
    delegateToUser: assign({ assignee: ({ event }) => event.userId }),
    retryWithBackoff: assign({ retryCount: ({ context }) => context.retryCount + 1 })
  },
  guards: {
    isFinalValidator: ({ context }) => context.validatorChain.length === context.currentIndex + 1,
    hasNextValidator: ({ context }) => context.validatorChain.length > context.currentIndex + 1,
    isSignedSuccessfully: ({ event }) => event.status === 'SIGNED'
  },
  actors: { callESignAdapter, callGEDAdapter }
}).createMachine({
  id: 'dossier',
  initial: 'draft',
  context: ({ input }) => ({ ...input, currentIndex: 0, retryCount: 0 }),
  states: {
    draft: { on: { SUBMIT: 'pending_validation' } },
    pending_validation: {
      entry: 'evaluateRules',
      on: {
        VALIDATE: [
          { target: 'pending_signature', guard: 'isFinalValidator' },
          { target: 'pending_validation', guard: 'hasNextValidator', actions: 'assignNext' }
        ],
        REJECT: 'rejected',
        DELEGATE: { actions: 'delegateToUser' }
      }
    },
    pending_signature: {
      invoke: { src: 'callESignAdapter', onDone: 'awaiting_callback', onError: 'sign_error' }
    },
    awaiting_callback: {
      on: {
        SIGN_CALLBACK: [
          { target: 'signed', guard: 'isSignedSuccessfully' },
          { target: 'rejected' }
        ]
      },
      after: { 86400000: 'expired' }  // 24h timeout
    },
    signed: { invoke: { src: 'callGEDAdapter', onDone: 'archived' } },
    archived: { type: 'final' },
    rejected: { type: 'final' },
    expired: { type: 'final' },
    sign_error: {
      after: { 60000: { target: 'pending_signature', actions: 'retryWithBackoff' } }
    }
  }
});
```

### Annexe B — Extrait OpenAPI socle (parapheur API)

```yaml
openapi: 3.0.3
info:
  title: Parapheur Digital API
  version: 1.0.0
  description: API interne banque pour le parapheur digital
servers:
  - url: https://parapheur.banque.ma/api/v1
security:
  - bearerAuth: []
paths:
  /dossiers:
    get:
      summary: Liste des dossiers
      parameters:
        - { name: page, in: query, schema: { type: integer, default: 1 } }
        - { name: pageSize, in: query, schema: { type: integer, default: 20, maximum: 100 } }
        - { name: statut, in: query, schema: { $ref: '#/components/schemas/StatutDossier' } }
        - { name: q, in: query, schema: { type: string }, description: 'Recherche full-text' }
      responses:
        '200':
          content:
            application/json:
              schema:
                type: object
                properties:
                  data: { type: array, items: { $ref: '#/components/schemas/Dossier' } }
                  pagination: { $ref: '#/components/schemas/Pagination' }
    post:
      summary: Créer un dossier
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/DossierCreate' }
      responses:
        '201': { content: { application/json: { schema: { $ref: '#/components/schemas/Dossier' } } } }

  /dossiers/{id}/submit:
    post:
      summary: Soumettre le dossier au workflow
      responses:
        '202': { description: Soumis }

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
  schemas:
    StatutDossier:
      type: string
      enum: [draft, pending_validation, pending_signature, signed, archived, rejected, cancelled]
    Dossier:
      type: object
      required: [id, reference, statut, montant, devise, createdAt]
      properties:
        id: { type: string, format: uuid }
        reference: { type: string }
        type_decision: { type: string }
        montant: { type: number }
        devise: { type: string, enum: [MAD, EUR, USD] }
        statut: { $ref: '#/components/schemas/StatutDossier' }
        createdAt: { type: string, format: date-time }
        # ...
    DossierCreate:
      type: object
      required: [type_decision, montant, devise, metadata]
      properties:
        type_decision: { type: string }
        montant: { type: number, minimum: 0 }
        devise: { type: string }
        metadata: { type: object }
    Pagination:
      type: object
      properties:
        page: { type: integer }
        pageSize: { type: integer }
        total: { type: integer }
```

### Annexe C — ADR types (Architecture Decision Records)

| ID | Titre | Statut |
|---|---|---|
| ADR-001 | Choix Node.js + Fastify + TypeScript pour le backend | Accepté |
| ADR-002 | State machine custom (XState) au lieu de Camunda | Accepté |
| ADR-003 | PostgreSQL full-text au lieu d'Elasticsearch | Accepté |
| ADR-004 | BullMQ + PG NOTIFY au lieu de Kafka | Accepté |
| ADR-005 | Hébergement DC Maroc Tier III+ (à confirmer cloud banque) | À valider P0 |
| ADR-006 | mTLS pour intégrations bancaires sortantes | Accepté |
| ADR-007 | Niveau signature SEA par défaut, SEQ optionnelle > 5 M MAD | À valider P0 (agrément DGSSI) |
| ADR-008 | Pas de Blue/Green Canary — rolling update + feature flags | Accepté |
| ADR-009 | Adapter pattern (ACL) pour intégrations e-sign et GED | Accepté |
| ADR-010 | i18n i18next + RTL Tailwind pour FR/AR | Accepté |
| ADR-011 | Audit log append-only avec hash-chain | Accepté |
| ADR-012 | Auth AD + MFA OTP banque (pas Okta) | Accepté |

### Annexe D — Glossaire technique

| Terme | Définition |
|---|---|
| **ACL** | Anti-Corruption Layer — pattern d'isolation entre modèles internes et externes |
| **ADR** | Architecture Decision Record |
| **CQRS** | Command Query Responsibility Segregation |
| **DDD** | Domain-Driven Design |
| **HSM** | Hardware Security Module |
| **JWT** | JSON Web Token |
| **mTLS** | Mutual TLS — authentification mutuelle client + serveur |
| **OWASP** | Open Web Application Security Project |
| **PWA** | Progressive Web App |
| **RBAC** | Role-Based Access Control |
| **ABAC** | Attribute-Based Access Control |
| **RPO** | Recovery Point Objective |
| **RTO** | Recovery Time Objective |
| **SAGA** | Pattern de transactions distribuées avec compensation |
| **STRIDE** | Spoofing, Tampering, Repudiation, Info disclosure, DoS, Elevation |
| **TLS** | Transport Layer Security |
| **WORM** | Write Once Read Many |
| **XState** | Bibliothèque JavaScript de machines à états |

---

*Fin de la Phase 4 v2 — Architecture & intégration.*

**Prochaine livraison** : Phase 2 v2 — Spécifications métier (règles DMN MAD, business case, ROI).
