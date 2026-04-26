# PHASE 4 — SPÉCIFICATIONS ARCHITECTURE & INTÉGRATION
## Parapheur Digital — Architecture applicative détaillée, Intégration ERP, Plan de déploiement & Migration

**Référence** : PARAPH-PH4-ARCH-INT-v1.0
**Statut** : Pour validation COPIL Architecture / DSI / Direction SI / Urbanisation
**Date** : 2026-04-25
**Auteur** : Direction Programme Parapheur (rôle McKinsey Senior Manager)
**Destinataires** : COPIL Direction (DSI, RSSI, Architecture d'Entreprise, Urbanisation, DAF)
**Niveau de classification** : C2 — Interne Restreint
**Pages** : ~90 équivalent A4

---

## SYNTHÈSE EXÉCUTIVE (Pyramide de Minto)

### Message principal (Top)
**Le parapheur digital s'intègre dans un écosystème SI complexe (SAP S/4HANA + 12 applications satellites + 18 référentiels maîtres) et doit traiter 12 000 dossiers/an avec un SLA de disponibilité de 99,9%. L'architecture cible — 8 microservices Java 21 + React 18, hébergés sur AWS Paris/SecNumCloud — représente un investissement de 685 K€ CAPEX et 285 K€/an OPEX, mais offre un TCO 5 ans inférieur de 1,4 M€ à une solution monolithique on-premises. Le plan de migration en 3 vagues sur 18 semaines minimise le risque opérationnel.**

### 5 décisions structurantes à arbitrer en COPIL

| # | Décision | Option recommandée | Coût | Délai | Risque si refus |
|---|----------|-------------------|------|-------|-----------------|
| **D1** | Pattern d'intégration SAP | **Event-driven (Kafka) pour async + REST/OData pour sync** | 95 K€ setup | T+10 sem | Couplage fort + indispo SAP = parapheur HS (perte 35K€/h indispo) |
| **D2** | Stratégie déploiement | **Blue/Green + Canary 5%→100% sur EKS Kubernetes** | +25% infra (justifié) | T+12 sem | Indispo lors releases (4h × 12 = 48h/an = 525 K€ perte) |
| **D3** | Reprise existant (legacy paper) | **Pas de migration historique — coexistence 18 mois puis archivage** | 45 K€ vs 320 K€ migration | T+18 sem | Si migration : risque corruption données + budget x7 |
| **D4** | Multi-région DR | **Active/Passive Paris (eu-west-3) → Francfort (eu-central-1)** | 85 K€ setup + 65 K€/an | T+14 sem | Sinistre majeur Paris = indispo 7+ jours (perte > 4 M€) |
| **D5** | Stratégie API publique | **Internal-only T+0, public B2B partner via API Manager T+12 mois** | 0€ MVP | T+18 sem | Si public direct : surface attaque x4 + besoin gouvernance API mature |

### Justification synthétique
1. **Event-driven SAP** : Kafka Connect SAP + IDoc → résilience aux indispos SAP (taux 99,5% historique → 4h/mois indispo cumulée). Async = parapheur continue à fonctionner.
2. **Blue/Green Canary** : déploiement sans interruption + rollback <2 min = -98% risque incident release vs déploiement direct.
3. **Pas de migration legacy** : 100K dossiers papier × coût migration 3,2€/dossier = 320 K€ pour zéro valeur métier (tous engagements clos). Coexistence + archive = 45 K€.
4. **DR Active/Passive** : RTO 4h, RPO 15min, conforme exigences DAF (assurance cyber).
5. **API privée d'abord** : maturité gouvernance avant exposition externe (10% des projets API publics réussissent en <12 mois selon Gartner).

### Budget consolidé
- **Architecture & Build** : 425 K€ (microservices, DB, Kafka, monitoring)
- **Intégration SI** : 165 K€ (SAP, Okta, Universign, 9 systèmes)
- **Déploiement & Migration** : 95 K€ (EKS, blue/green, DR setup)
- **TOTAL CAPEX** : **685 K€** sur 18 semaines
- **OPEX récurrent** : 285 K€/an (cloud, licences, supervision)
- **TCO 5 ans cible** : 2,1 M€ (vs 3,5 M€ alternative monolithique on-prem)

---

## SOMMAIRE

- [Section A — Architecture applicative détaillée (C4 + microservices)](#section-a)
- [Section B — Intégration SI & ERP (SAP + 12 systèmes)](#section-b)
- [Section C — Architecture infrastructure & cloud](#section-c)
- [Section D — Plan de déploiement & stratégie release](#section-d)
- [Section E — Plan de migration & coexistence legacy](#section-e)
- [Section F — Disaster Recovery, Continuité, Observabilité](#section-f)
- [Section G — Registre des risques architecture (P×I)](#section-g)
- [Section H — Plan d'implémentation, budget, GO/NO-GO](#section-h)
- [Annexes — Diagrammes, OpenAPI, Runbooks](#annexes)

---

<a name="section-a"></a>
## SECTION A — ARCHITECTURE APPLICATIVE DÉTAILLÉE

### A.1 Vue C4 niveau 1 — Contexte système

```
┌──────────────────────────────────────────────────────────────────────┐
│                      ÉCOSYSTÈME PARAPHEUR DIGITAL                    │
│                                                                      │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐     │
│  │ Initiat. │     │ Validat. │     │ Signat.  │     │ Admin    │     │
│  │ (2400)   │     │ (615)    │     │ (85)     │     │ (15)     │     │
│  └────┬─────┘     └────┬─────┘     └────┬─────┘     └────┬─────┘     │
│       └────────────────┴────────────────┴────────────────┘            │
│                              │                                       │
│                       ┌──────▼──────┐                                │
│                       │  PARAPHEUR  │                                │
│                       │   DIGITAL   │ ←──── Système central          │
│                       └──────┬──────┘                                │
│                              │                                       │
│      ┌───────────┬───────────┼───────────┬───────────┐               │
│      │           │           │           │           │               │
│  ┌───▼───┐  ┌────▼────┐ ┌────▼───┐  ┌────▼────┐ ┌────▼────┐          │
│  │ SAP   │  │ Okta    │ │Universign │  │ SAP HR  │ │Splunk  │        │
│  │S/4HANA│  │ (IDP)   │ │ (PSCo)    │  │(Workday)│ │ (SIEM) │        │
│  └───────┘  └─────────┘ └───────┘  └─────────┘ └─────────┘           │
│  + 7 autres systèmes (Salesforce, ServiceNow, Coupa, Concur,        │
│    BO Reporting, eDoc archive, eFax)                                 │
└──────────────────────────────────────────────────────────────────────┘
```

**Acteurs externes** :

| Acteur | Type | Rôle | Volumétrie |
|--------|------|------|------------|
| Initiateurs | Humain | Créent dossiers | ~2400 users, ~50 actions/j |
| Validateurs | Humain | Approuvent workflow | ~615 users, ~30 actions/j |
| Signataires | Humain | Signent documents | ~85 users, ~10 actions/j |
| Admin métier/IT | Humain | Configurent règles, supervisent | ~15 users |
| SAP S/4HANA | Système | ERP central — référentiels + écritures | 50K appels/j |
| SAP HR (Workday futur) | Système | Source identités/orga | 5K appels/j |
| Okta | Système | IDP SSO + MFA | 10K auth/j |
| Universign | Système | PSCo signature qualifiée | ~50 sign/j |
| Salesforce | Système | CRM clients/contrats | 500 appels/j |
| ServiceNow | Système | ITSM tickets / incidents | 100/j |
| Coupa | Système | Achats P2P (alternative SAP MM) | 800/j |
| Concur | Système | Notes de frais | 200/j |
| BO Reporting | Système | Reporting consolidé | Batch nightly |
| eDoc archive | Système | Archivage à valeur probante (10 ans+) | Push doc signés |
| Splunk | Système | SIEM centralisé | 100M events/j |
| eFax | Système | Pour partenaires sans email | 50/j |

### A.2 Vue C4 niveau 2 — Containers (8 microservices)

```
┌────────────────────────────────────────────────────────────────────────┐
│                      PARAPHEUR DIGITAL — Containers                    │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │                    Frontend (React 18 + TS)                      │  │
│  │  - SPA Vite + TanStack Query + React Router                      │  │
│  │  - Design System (shadcn/ui + Tailwind)                          │  │
│  │  - 23 écrans, PWA, offline-first lecture                         │  │
│  └────────────────────────────┬─────────────────────────────────────┘  │
│                               │ HTTPS / TLS 1.3                        │
│  ┌────────────────────────────▼─────────────────────────────────────┐  │
│  │              API Gateway (Kong Gateway 3.x)                      │  │
│  │  - Auth JWT + OAuth 2.0 + rate limiting + WAF                    │  │
│  │  - Routing vers microservices, observability                     │  │
│  └─┬────────┬────────┬────────┬────────┬────────┬────────┬─────────┘  │
│    │        │        │        │        │        │        │            │
│  ┌─▼────┐┌──▼─────┐┌─▼──────┐┌▼───────┐┌▼────┐┌─▼─────┐┌─▼───────┐    │
│  │MS-1  ││MS-2    ││MS-3    ││MS-4    ││MS-5 ││MS-6   ││MS-7     │    │
│  │Dossier││Workflow││Signat. ││Notify  ││Audit ││Référ.  ││Reporting│   │
│  │(Java) ││(Camunda)││(Java) ││(Java)  ││(Java)││(Java) ││(Java)   │    │
│  └─┬────┘└──┬─────┘└─┬──────┘└┬───────┘└┬────┘└─┬─────┘└─┬───────┘    │
│    │       │         │        │         │       │        │            │
│  ┌─▼───────▼─────────▼────────▼─────────▼───────▼────────▼─────────┐  │
│  │              MS-8 Integration Bus (Kafka 3.7 + Connect)         │  │
│  │  - Events workflow + sync SAP/HR/Salesforce                     │  │
│  └──────────────────────────────┬─────────────────────────────────┘  │
│                                 │                                    │
│  ┌──────────────────────────────▼──────────────────────────────────┐  │
│  │                       DATA LAYER                                │  │
│  │  PostgreSQL 16 (RDS Multi-AZ)  │  Redis 7 (ElastiCache)          │  │
│  │  S3 + KMS (documents)          │  Elasticsearch 8 (search/audit) │  │
│  │  Vault (secrets)               │  HSM Thales (clés QES)          │  │
│  └─────────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────┘
```

### A.3 Microservices — Spécifications détaillées

#### MS-1 : Service Dossier (Document Service)

**Responsabilités** :
- CRUD dossier (création, modification, consultation, archivage)
- Gestion documents PDF (upload, version, hash)
- Assignation type dossier → workflow

**Stack** :
- Java 21 + Spring Boot 3.2
- PostgreSQL : tables `dossier`, `dossier_document`, `dossier_metadata`
- S3 : stockage PDF (`s3://parapheur-prod/{filiale}/dossiers/{year}/{dossier_id}/`)

**API REST principales** :
```yaml
POST   /api/v1/dossiers                    # Créer
GET    /api/v1/dossiers/{id}               # Lire
PUT    /api/v1/dossiers/{id}               # Modifier (draft only)
GET    /api/v1/dossiers?filters=...        # Lister
POST   /api/v1/dossiers/{id}/submit        # Soumettre workflow
POST   /api/v1/dossiers/{id}/cancel        # Annuler (initiateur)
GET    /api/v1/dossiers/{id}/documents     # Lister documents
POST   /api/v1/dossiers/{id}/documents     # Upload PDF
GET    /api/v1/dossiers/{id}/documents/{doc_id}  # Télécharger
```

**Events publiés (Kafka)** :
- `dossier.created.v1`
- `dossier.submitted.v1`
- `dossier.cancelled.v1`
- `dossier.completed.v1`

**Volumétrie** : 12 K dossiers/an, ~33/j moyenne, pic 240/j fin de mois.

**Performance cible** : P95 < 300ms, P99 < 800ms.

#### MS-2 : Service Workflow (Camunda Engine)

**Responsabilités** :
- Orchestration workflow BPMN 2.0
- Décisions DMN (87 règles métier Phase 2)
- Gestion états + tâches utilisateur
- SLA tracking + escalades

**Stack** :
- Camunda Platform 7 Enterprise
- Java 21 + Spring Boot 3.2 (embed Camunda)
- PostgreSQL (Camunda schema)

**Modèles BPMN déployés** : 6 (Achat, RH, Contrat, Capex, Juridique, Marketing — cf. Phase 2)

**Tables DMN** : 87 tables (53 principales + 34 modificateurs)

**API principales** :
```yaml
POST   /api/v1/workflow/instances          # Démarrer workflow
GET    /api/v1/workflow/instances/{id}     # État instance
GET    /api/v1/workflow/tasks?assignee={user}  # Mes tâches
POST   /api/v1/workflow/tasks/{id}/complete    # Compléter tâche
POST   /api/v1/workflow/tasks/{id}/delegate    # Déléguer
GET    /api/v1/workflow/decisions/evaluate     # Évaluer DMN
```

**Events** :
- `workflow.started.v1`, `workflow.task.created.v1`, `workflow.task.completed.v1`
- `workflow.escalated.v1`, `workflow.completed.v1`

**Performance** : P95 démarrage instance < 500ms, évaluation DMN < 50ms.

#### MS-3 : Service Signature

**Responsabilités** :
- Orchestration signature AdES + QES
- Intégration Universign API
- Apposition PAdES-LTA + horodatage TSA
- Vérification signatures long terme

**Stack** :
- Java 21 + Spring Boot
- Bouncy Castle 1.78 (crypto)
- Apache PDFBox 3.0 (manipulation PDF)
- iText 8 Enterprise (PAdES advanced)

**API** :
```yaml
POST   /api/v1/signatures/prepare          # Préparer doc à signer (hash)
POST   /api/v1/signatures/sign             # Demander signature
GET    /api/v1/signatures/{id}/status      # Statut Universign
POST   /api/v1/signatures/verify           # Vérifier signature
GET    /api/v1/signatures/{id}/proof       # Dossier de preuve
```

**Intégration Universign** :
- API REST v3 + webhook callback
- Auth OAuth 2.0 client credentials
- Modes : AdES, QES, dual (AdES + QES)

#### MS-4 : Service Notification

**Responsabilités** :
- Email (SMTP via SendGrid)
- SMS (Twilio)
- Notifications in-app (WebSocket)
- Push mobile (FCM/APNS)
- Templates multi-langue (FR/EN/DE)

**Stack** :
- Java 21 + Spring Boot
- Thymeleaf (templating email)
- Redis (queue notifications + déduplication)

**Patterns** :
- Async via Kafka consumer
- Retry exponentiel (3 tentatives)
- Dead letter queue pour échecs persistants

**Volumétrie** : ~80 K notifications/an (35 K email + 5 K SMS + 40 K in-app).

#### MS-5 : Service Audit

**Responsabilités** :
- Journal immuable de toutes actions métier
- Export forensique (RGPD, audit, contentieux)
- Recherche full-text Elasticsearch

**Stack** :
- Java 21 + Spring Boot
- PostgreSQL (table `audit_log` + trigger anti-modification)
- Elasticsearch 8 (indexation pour recherche)
- S3 archive (>1 an)

**Schéma audit** :
```sql
CREATE TABLE audit_log (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    user_id VARCHAR(50) NOT NULL,
    user_role VARCHAR(50),
    filiale_code VARCHAR(10),
    action VARCHAR(100) NOT NULL,
    entity_type VARCHAR(50),
    entity_id UUID,
    old_value JSONB,
    new_value JSONB,
    ip_address INET,
    user_agent TEXT,
    correlation_id UUID,
    hash_sha512 VARCHAR(128),    -- Hash auto = (prev_hash + content)
    prev_log_id UUID              -- Chaînage hash pour intégrité
) PARTITION BY RANGE (timestamp);

-- Trigger empêche UPDATE/DELETE
CREATE TRIGGER audit_log_immutable
BEFORE UPDATE OR DELETE ON audit_log
FOR EACH ROW EXECUTE FUNCTION raise_immutable_error();
```

**Volumétrie** : ~500 K events/an, partitioning mensuel, rétention 10 ans.

#### MS-6 : Service Référentiels (Master Data Service)

**Responsabilités** :
- Synchronisation référentiels SAP MDG → cache local
- 17 référentiels (cf. Phase 2 §C.2)
- Recherche/typeahead utilisateur

**Stack** :
- Java 21 + Spring Boot
- PostgreSQL + Redis (cache 5 min TTL)
- Elasticsearch (recherche)

**Patterns sync** :
- **Event-driven** (Kafka) : SAP MDG publie changement → MS-6 consomme → MAJ cache
- **Fallback batch** quotidien (3h du matin) : full sync
- **Cache invalidation** : Redis pub/sub vers tous nodes

**Référentiels gérés** :

| Réf. | Volumétrie | Source | Sync |
|------|-----------|--------|------|
| Sociétés | 18 | SAP | Daily batch |
| Filiales | 35 | SAP | Daily batch |
| Centres coût | 850 | SAP | Hourly |
| Centres profit | 120 | SAP | Hourly |
| Comptes GL | 1200 | SAP | Daily |
| Fournisseurs | 8500 | SAP MDG | Real-time |
| Clients | 4200 | Salesforce | Real-time |
| Articles | 25000 | SAP | Daily |
| Devises + taux | 15 + EOD | SAP / Banque France | Daily |
| Salariés | 3000 | SAP HR / Workday | Real-time |
| Organisation | 450 nœuds | SAP HR | Daily |
| Rôles parapheur | 9 | Local config | Manual |
| Délégations | ~50 actives | Local | Real-time |
| Types dossier | 6 + sous-types | Local config | Manual |
| Workflows BPMN | 6 | Camunda | Manual deploy |
| Règles DMN | 87 | Camunda | Manual deploy |
| Templates email | 35 | Local config | Manual |

#### MS-7 : Service Reporting

**Responsabilités** :
- Dashboards opérationnels (KPI temps réel)
- Reports périodiques (CSV, PDF, Excel)
- Datamart vers BO/PowerBI

**Stack** :
- Java 21 + Spring Boot
- PostgreSQL (réplicas read-only)
- Apache Superset (dashboards self-service)
- Datamart PostgreSQL alimenté CDC (Debezium)

**KPI temps réel** (cf. Phase 1 §C.4) :
- Délai moyen workflow par filiale/type
- Taux conformité SLA
- Volume signatures (AdES vs QES)
- Top 10 validateurs en retard
- Taux rejet par étape

#### MS-8 : Integration Bus (Kafka)

**Responsabilités** :
- Bus événementiel inter-microservices
- Connecteurs Kafka Connect vers SAP/Salesforce/Workday
- Schema Registry (Avro/Protobuf)

**Stack** :
- Apache Kafka 3.7 (managed AWS MSK)
- Kafka Connect (SAP S/4HANA Connector + Salesforce Connector)
- Confluent Schema Registry
- Kafka Streams (transformations event)

**Topics principaux** :

| Topic | Producer | Consumers | Volume/j | Rétention |
|-------|----------|-----------|----------|-----------|
| `parapheur.dossier.events.v1` | MS-1 | MS-2, MS-5, MS-7 | ~50 | 7j |
| `parapheur.workflow.events.v1` | MS-2 | MS-4, MS-5, MS-7 | ~200 | 7j |
| `parapheur.signature.events.v1` | MS-3 | MS-5, MS-7, eDoc | ~50 | 30j |
| `parapheur.notification.requests.v1` | MS-1,2,3 | MS-4 | ~300 | 3j |
| `sap.master.data.changes.v1` | SAP Connect | MS-6 | ~500 | 30j |
| `sap.financial.postings.v1` | MS-2 (signed) | SAP Connect | ~50 | 30j |
| `audit.events.v1` | All MS | MS-5 | ~5000 | 90j |

### A.4 Patterns architecture

#### A.4.1 Saga pattern (transactions distribuées)

**Use case** : signature dossier achat 75K€ → écriture comptable SAP.

```
1. MS-2 Workflow valide signature
2. MS-3 Service signature : appose QES
3. MS-8 publie event `signature.completed`
4. SAP Connect consomme → tente écriture FI
5. Si succès : event `sap.posting.created` → MS-1 update statut "POSTED"
6. Si échec : compensation = event `signature.compensation.required`
   → MS-3 marque dossier "POSTED_FAILED" (manuel intervention)
   → Notification CFO
```

#### A.4.2 CQRS (Command/Query Responsibility Segregation)

- **Write side** : MS-1, MS-2, MS-3 (PostgreSQL primary)
- **Read side** : MS-7 Reporting (réplicas + Elasticsearch + datamart)
- **Synchronisation** : Debezium CDC (Change Data Capture)

#### A.4.3 Circuit Breaker (Resilience4j)

Tous appels externes (SAP, Universign, Okta) protégés par :
- Threshold : 50% erreurs sur 20 calls → ouvre 30s
- Fallback : queue Kafka retry + alerte SOC

#### A.4.4 Idempotency

Tous endpoints `POST` critiques requièrent header `Idempotency-Key` (UUID v4) :
- Cache 24h Redis
- Re-jeu retourne réponse originale (anti-double facturation, anti-double signature)

### A.5 Modèle de données global

**18 tables principales** (cf. Phase 2 §C pour dictionnaire 142 attributs) :

```
dossier (PK: id UUID)
├── dossier_document (FK dossier_id) — versions PDF
├── dossier_metadata (FK dossier_id) — attributs métier dynamiques
├── dossier_workflow_instance (FK dossier_id, camunda_process_instance_id)
├── dossier_signature (FK dossier_id) — signatures AdES/QES
├── dossier_audit (FK dossier_id) — événements
└── dossier_attachment (FK dossier_id) — pièces jointes

workflow_task (Camunda) — tâches utilisateur
workflow_decision (Camunda) — décisions DMN

user (synced from SAP HR)
├── user_role (FK user_id)
├── user_delegation (FK delegant, delegataire)
└── user_habilitation (FK user_id)

referentiel_filiale, referentiel_centre_cout, referentiel_compte, ...
(17 tables référentiels)

audit_log (immutable, partitioned monthly)
notification (FK user_id) — envois trace
```

**Sizing PostgreSQL** :
- Année 1 : ~50 GB (12K dossiers × ~4 MB moyennes documents inclus)
- Documents PDF dans S3, BDD = métadonnées seulement → ~5 GB BDD pure
- Année 5 : ~25 GB BDD + 50 GB index Elasticsearch + 10 TB S3
- Croissance : ~5 GB BDD/an + 2 TB S3/an

---

<a name="section-b"></a>
## SECTION B — INTÉGRATION SI & ERP

### B.1 Cartographie d'intégration globale

```
┌────────────────────────────────────────────────────────────────────────┐
│                    PARAPHEUR DIGITAL                                   │
└────────────────────────┬───────────────────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┬────────────────┐
        │                │                │                │
   ┌────▼──────┐  ┌──────▼───────┐  ┌─────▼─────┐  ┌───────▼────────┐
   │ SYNCHRONE │  │ ASYNCHRONE   │  │ FILE      │  │ INTERACTIF     │
   │ REST/SOAP │  │ EVENT KAFKA  │  │ TRANSFER  │  │ UI EMBEDDED    │
   └────┬──────┘  └──────┬───────┘  └─────┬─────┘  └───────┬────────┘
        │                │                │                │
   ┌────▼──────────────────────────────────────────────────▼─────┐
   │  SAP S/4HANA  │  Workday  │  Salesforce  │  Universign       │
   │  ServiceNow   │  Coupa    │  Concur      │  BO/PowerBI       │
   │  Okta         │  eDoc     │  Splunk      │  eFax             │
   └──────────────────────────────────────────────────────────────┘
```

**13 systèmes intégrés**, 4 patterns d'intégration.

### B.2 Intégration SAP S/4HANA (cœur ERP)

**Statut SAP** : on-premises actuellement, migration RISE with SAP prévue 2027.

**Périmètre intégration** :

| Module SAP | Type intégration | Direction | Volume | Pattern |
|-----------|------------------|-----------|--------|---------|
| **MDG** (Master Data Governance) | Sync référentiels | SAP → Parapheur | Real-time + daily batch | Event-driven Kafka Connect |
| **FI** (Financial) | Écriture comptable post-signature | Parapheur → SAP | ~30/j | IDoc async + Kafka |
| **MM** (Materials Mgmt) | Création commande achat | Parapheur → SAP | ~25/j | OData + IDoc |
| **HR** (HCM) | Identités, organisation | SAP → Parapheur | Real-time + daily | SCIM + Event |
| **PS** (Project System) | Capex, projets | Parapheur → SAP | ~3/j | OData |
| **CO** (Controlling) | Centres coût/profit | SAP → Parapheur | Daily | OData read |

#### B.2.1 Pattern event-driven (Kafka Connect SAP)

**Connecteur** : Confluent SAP S/4HANA Connector (officiel)

**Topics SAP → Parapheur** :
- `sap.bp.created`, `sap.bp.changed` (Business Partner = fournisseurs/clients)
- `sap.cost_center.changed`
- `sap.gl_account.changed`
- `sap.material.changed`

**Topics Parapheur → SAP** :
- `parapheur.fi.posting.request` → connector convertit en BAPI_ACC_DOCUMENT_POST
- `parapheur.mm.po.create.request` → BAPI_PO_CREATE1
- `parapheur.ps.budget.commit.request` → BAPI_BUS2001_CREATE

#### B.2.2 Pattern synchrone REST (OData v4)

**Pour lookups temps réel** (UI initiateur recherche fournisseur) :
```
GET https://sap-api.parapheur.com/sap/opu/odata4/sap/api_business_partner/srvd_a2x/sap/business_partner/0001/BusinessPartner?$filter=BusinessPartnerCategory eq '2' and contains(BusinessPartnerFullName, 'ACME')&$top=10
```

**Sécurité** :
- mTLS entre parapheur ↔ SAP API Gateway
- OAuth 2.0 Client Credentials (technical user `WS_PARAPHEUR_TECH`)
- Scope minimum : `BP_READ`, `COST_CENTER_READ`

#### B.2.3 Pattern IDoc asynchrone (writes critiques)

**Pour écritures FI/MM** (où SAP doit valider règles métier complexes) :
```
1. Parapheur publie sur Kafka topic
2. Kafka Connect SAP convertit message → IDoc (FIDCMT01, ORDERS05, etc.)
3. IDoc envoyé via tRFC vers SAP
4. SAP traite IDoc → status 53 (success) ou 51 (error)
5. SAP publie ACK back via IDoc → Parapheur consomme
6. Parapheur update statut dossier "POSTED" ou "POSTING_FAILED"
```

**Avantages** :
- Découplage : SAP indispo n'impacte pas parapheur
- Replay possible si erreur (Kafka rétention 30j)
- Traçabilité complète

**Risques & mitigation** :
- Latence : minutes vs secondes synchrone → acceptable pour FI post-signature
- Écritures dupliquées : idempotency key dans IDoc header

### B.3 Intégration Okta (IAM/SSO)

**Patterns** :

| Flow | Protocole | Cas d'usage |
|------|-----------|-------------|
| Web SSO | SAML 2.0 | Login UI parapheur |
| Mobile auth | OIDC + PKCE | App mobile native |
| API tiers | OAuth 2.0 Client Credentials | SAP, Salesforce → Parapheur |
| Provisioning | SCIM 2.0 | SAP HR → Okta → Parapheur (création comptes) |
| MFA step-up | Okta Verify + WebAuthn | Signature QES |

**Configuration cible** :
- Application Okta : "Parapheur Digital" (SAML SP)
- Group rules : auto-assign role basé sur attribut SAP HR `cost_center`
- Sign-on policy :
  - Standard : SSO + MFA TOTP
  - Privileged action : SSO + MFA WebAuthn
  - Service account : 2FA + IP whitelist

### B.4 Intégration Universign (signature QES)

**API utilisée** : Universign API v3 (REST)

**Flows critiques** :

#### B.4.1 Initialisation signature

```http
POST https://ws.universign.eu/sign/rest/v3/transactions/
Authorization: Bearer {oauth_token}
Content-Type: application/json

{
  "profile": "default",
  "signers": [{
    "firstname": "Marie",
    "lastname": "Dupont",
    "email": "marie.dupont@societe.com",
    "phoneNum": "+33612345678",
    "certificateType": "qualified"
  }],
  "documents": [{
    "name": "Contrat-2026-0042.pdf",
    "content": "{base64_pdf}",
    "signatureFields": [{
      "page": 5,
      "x": 350, "y": 100,
      "width": 200, "height": 50
    }]
  }],
  "successUrl": "https://parapheur.com/sign/success",
  "cancelUrl": "https://parapheur.com/sign/cancel",
  "failUrl": "https://parapheur.com/sign/fail",
  "callbackUrl": "https://api.parapheur.com/v1/signatures/webhook"
}
```

#### B.4.2 Webhook callback (signature complétée)

```http
POST /api/v1/signatures/webhook
X-Universign-Signature: sha256={hmac}

{
  "transactionId": "TX-7e8f9a0b",
  "status": "COMPLETED",
  "completedAt": "2026-04-25T14:32:18Z",
  "signers": [{
    "id": "SIG-123",
    "status": "SIGNED",
    "certificateId": "CERT-456",
    "evidenceTrail": "https://ws.universign.eu/.../evidence.pdf"
  }],
  "signedDocuments": [{
    "name": "Contrat-2026-0042-SIGNED.pdf",
    "downloadUrl": "https://ws.universign.eu/.../download/{token}"
  }]
}
```

**Vérification webhook** : HMAC-SHA256 avec secret partagé (anti-spoofing).

### B.5 Intégration Salesforce (CRM)

**Cas d'usage** :
- Initiation dossier "Contrat Client" → autocomplete client depuis Salesforce
- Post-signature contrat → MAJ opportunité Salesforce status "Won"

**Pattern** :
- Salesforce Connect (data virtualization) : pas de duplication, lookup temps réel
- API REST Salesforce v60 + JWT Bearer Flow

### B.6 Intégration Workday (HR — futur 2027)

**Migration** : SAP HR → Workday prévue 2027.

**Stratégie de transition** :
- Phase 1 (T+0 à T+6 mois) : SAP HR comme source
- Phase 2 (2027) : adapter SCIM provisioning Workday → Okta → Parapheur
- Adaptation MS-6 référentiel salariés : connecteur Workday Reports-as-a-Service

### B.7 Intégration eDoc (archivage probant)

**Système cible** : Locarchives (SAE qualifié NF Z42-013 + ISO 14641-1)

**Flow post-signature** :
```
1. MS-3 Signature → event "signature.completed"
2. MS-8 Kafka → connector eDoc
3. Push doc signé + métadonnées → eDoc API REST
4. eDoc retourne identifiant pérenne (CDC = Code Document Conservé)
5. MS-1 store CDC dans dossier_metadata
6. eDoc gère ensuite : conservation 10+ ans, scellement, restauration
```

**Avantage** : SAE externe = preuve probante renforcée (jurisprudence stable) + délégation contrainte légale.

### B.8 Intégration ServiceNow (ITSM)

**Cas** : tickets incidents/anomalies remontés depuis parapheur.

**Pattern** : webhook ServiceNow + REST API

```
Erreur grave parapheur → MS-5 Audit log → trigger → POST ServiceNow
{
  "category": "Application",
  "subcategory": "Parapheur",
  "priority": "2-High",
  "description": "Échec écriture SAP FI - dossier {id}",
  "caller_id": "PARAPHEUR_SYS"
}
```

### B.9 Sécurité intégrations

**Standards appliqués** :

| Mécanisme | Usage | Justification |
|-----------|-------|---------------|
| mTLS | Toutes APIs internes | Auth machine-to-machine forte |
| OAuth 2.0 + scopes | Toutes APIs | Standard industrie |
| JWT signé RS256 | Tokens utilisateurs | Vérifiable sans appel IDP |
| HMAC-SHA256 | Webhooks (Universign, ...) | Anti-spoofing |
| API Gateway WAF | Tous trafics externes | OWASP Top 10 |
| Rate limiting | Par tenant + global | Anti-DoS |
| IP whitelist | SAP, eDoc | Defense in depth |
| Vault dynamic secrets | DB credentials | Rotation auto |

---

<a name="section-c"></a>
## SECTION C — ARCHITECTURE INFRASTRUCTURE & CLOUD

### C.1 Choix cloud : AWS Paris (eu-west-3) — alternative SecNumCloud

#### C.1.1 Évaluation comparative

| Critère | AWS Paris | OVH SecNumCloud | Azure France | Outscale (Dassault) |
|---------|-----------|-----------------|--------------|---------------------|
| Souveraineté FR | Hébergé FR mais juridiction US (CLOUD Act) | **OUI 100%** | Hébergé FR juridiction US | OUI 100% |
| Qualification ANSSI | NON | **SecNumCloud** | NON | SecNumCloud |
| Maturité services | **Excellente** | Moyenne | Très bonne | Moyenne |
| Ecosystem managed | **Vaste** | Limité | Bon | Limité |
| Coût (équivalent) | Référence | +20% | -5% | +15% |
| Time to market | **Rapide** | Lent | Rapide | Moyen |
| Compétences équipe | **Existantes** | À former | Partielles | À former |

**Décision recommandée** :
- **MVP & majorité production** : AWS eu-west-3 (Paris)
- **Données sensibles secteur public client** : OVH SecNumCloud (cluster dédié)
- **Stratégie hybrid** : 90% AWS + 10% SecNumCloud (pour 28% CA secteur public)

### C.2 Architecture AWS détaillée

```
┌──────────────────────────────────────────────────────────────────────┐
│                       AWS Account : parapheur-prod                   │
│                       Region : eu-west-3 (Paris)                     │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │  VPC : 10.0.0.0/16  — Multi-AZ (3 AZ)                        │    │
│  │                                                              │    │
│  │  ┌──────────────────────────────────────────────────────┐    │    │
│  │  │  PUBLIC SUBNETS (3 × /24)                            │    │    │
│  │  │  - ALB (Application Load Balancer)                   │    │    │
│  │  │  - NAT Gateway (3, un par AZ)                        │    │    │
│  │  └─────────────────────┬────────────────────────────────┘    │    │
│  │                        │                                     │    │
│  │  ┌─────────────────────▼────────────────────────────────┐    │    │
│  │  │  PRIVATE SUBNETS - APP (3 × /23)                     │    │    │
│  │  │  - EKS Cluster (3 nodes m6i.xlarge per AZ)           │    │    │
│  │  │    - 8 microservices (Java 21 containers)            │    │    │
│  │  │    - Kong API Gateway                                │    │    │
│  │  │  - Frontend S3 + CloudFront (separate)               │    │    │
│  │  └─────────────────────┬────────────────────────────────┘    │    │
│  │                        │                                     │    │
│  │  ┌─────────────────────▼────────────────────────────────┐    │    │
│  │  │  PRIVATE SUBNETS - DATA (3 × /23)                    │    │    │
│  │  │  - RDS PostgreSQL 16 Multi-AZ (db.r6g.2xlarge)       │    │    │
│  │  │  - ElastiCache Redis 7 (cache.r6g.large × 3)         │    │    │
│  │  │  - MSK Kafka 3.7 (3 brokers × kafka.m5.large)        │    │    │
│  │  │  - OpenSearch (Elasticsearch) m6g.large × 3          │    │    │
│  │  └─────────────────────┬────────────────────────────────┘    │    │
│  │                        │                                     │    │
│  │  ┌─────────────────────▼────────────────────────────────┐    │    │
│  │  │  PRIVATE SUBNETS - SHARED                            │    │    │
│  │  │  - HSM CloudHSM (Thales) × 2                         │    │    │
│  │  │  - Vault HCP Enterprise (peering)                    │    │    │
│  │  │  - Direct Connect → SAP On-Prem                      │    │    │
│  │  └──────────────────────────────────────────────────────┘    │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Services régionaux :                                                │
│  - S3 (eu-west-3 + cross-region replication eu-central-1)           │
│  - KMS (CMK per filiale)                                             │
│  - Route 53 + CloudFront                                             │
│  - WAF v2 + Shield Standard                                          │
│  - Secrets Manager (fallback Vault)                                  │
│  - CloudTrail + Config + GuardDuty                                   │
└──────────────────────────────────────────────────────────────────────┘
```

### C.3 Sizing & dimensionnement

#### C.3.1 EKS workers

| Composant | Type instance | Quantité | Raison |
|-----------|--------------|----------|--------|
| App nodes | m6i.xlarge (4 vCPU, 16 GB) | 6 (2 × 3 AZ) | 8 microservices × 2 replicas + headroom |
| Camunda nodes | m6i.2xlarge (8 vCPU, 32 GB) | 3 | Workflow gourmand mémoire |
| Burst capacity | Cluster Autoscaler | 0-9 nodes | Pic fin de mois (×3 charge) |

**Total prod compute** : 9 nodes nominal × ~210€/mois = **1 890€/mois** (22,7K€/an)

#### C.3.2 RDS PostgreSQL

- Instance : db.r6g.2xlarge (8 vCPU, 64 GB RAM)
- Storage : 500 GB GP3 + auto-scaling jusqu'à 2 TB
- Backup : 35 jours, snapshots quotidiens
- Multi-AZ : OUI (failover ~1 min)
- Read replica : 1 (pour MS-7 Reporting)
- **Coût** : ~1 200€/mois × 2 (primary + replica) = 2 400€/mois (**28,8K€/an**)

#### C.3.3 Autres services managés

| Service | Sizing | Coût mensuel |
|---------|--------|--------------|
| ElastiCache Redis | 3 × cache.r6g.large | 480€ |
| MSK Kafka | 3 × kafka.m5.large | 720€ |
| OpenSearch | 3 × m6g.large + 200 GB | 540€ |
| S3 | 10 TB stockage + transferts | 280€ |
| CloudFront | 500 GB transfert | 45€ |
| Route 53 + ALB | - | 35€ |
| WAF v2 | 100M requests | 60€ |
| KMS | 50 keys | 50€ |
| CloudHSM | 2 instances | 1 700€ |
| GuardDuty + Config | - | 220€ |
| Data transfer | Egress estimé | 350€ |
| **TOTAL services** | - | **4 480€/mois** |

#### C.3.4 Coût AWS total prod

| Catégorie | €/mois | €/an |
|-----------|--------|------|
| Compute (EKS) | 1 890 | 22,7K |
| Database (RDS) | 2 400 | 28,8K |
| Services managés | 4 480 | 53,8K |
| **Sous-total prod** | **8 770** | **105,2K** |
| Env STAGING (50% prod) | 4 385 | 52,6K |
| Env DEV (20% prod) | 1 754 | 21,1K |
| **TOTAL AWS** | **14 909** | **178,9K** |

**+ DR multi-region** : ~35 K€/an (cf. §F.1).

### C.4 Réseau & sécurité périmétrique

```
Internet → Cloudflare (DDoS + WAF) → AWS WAF v2 → ALB → Kong → Microservices
                                                   ↓
                                             VPC Endpoints (S3, KMS, Secrets)
                                                   ↓
                                       Direct Connect → On-Prem (SAP, AD)
```

**Contrôles** :
- Cloudflare Enterprise : DDoS layer 3-7, bot management, rate limiting
- AWS WAF v2 : OWASP Top 10 managed rules + custom rules métier
- VPC Flow Logs → Splunk SIEM
- Network ACL + Security Groups (least privilege)
- AWS PrivateLink pour services AWS (pas d'egress internet inutile)

### C.5 Containerisation & orchestration

**Stratégie Docker** :
- Images base : `eclipse-temurin:21-jre-alpine` (JRE Java 21 minimal)
- Multi-stage build (builder + runtime)
- Non-root user (UID 1000)
- Read-only root filesystem
- Distroless si possible (réduction surface attaque)

**Manifests Kubernetes** :
- Helm Charts par microservice
- ArgoCD pour GitOps (manifests versionnés Git)
- HPA (Horizontal Pod Autoscaler) basé CPU + custom metrics (queue depth Kafka)
- PDB (Pod Disruption Budget) min 1 pod par service durant maintenance
- Network Policies (Calico) — communication explicite uniquement

**Sécurité runtime** :
- Falco (détection comportement anormal containers)
- OPA Gatekeeper (policies admission)
- ImagePolicyWebhook : seulement images signées Cosign

---

<a name="section-d"></a>
## SECTION D — PLAN DE DÉPLOIEMENT & STRATÉGIE RELEASE

### D.1 Stratégie release : Blue/Green + Canary

```
ÉTAT INITIAL : 100% trafic → BLUE (v1.0)

STEP 1 : Déploiement nouvelle version
  Deploy GREEN (v1.1) en parallèle, 0% trafic
  ↓
  Smoke tests automatiques GREEN (90 tests E2E)
  ↓
  Si KO → destruction GREEN, rollback immédiat
  Si OK → step 2

STEP 2 : Canary 5%
  Route 53 weighted : 95% BLUE / 5% GREEN
  ↓
  Monitoring 1h (Datadog dashboards)
  - Error rate < 0,5%
  - P95 latency dégradation < 10%
  - Business KPI stable (signatures réussies)
  ↓
  Si KO → rollback automatique (bascule 100% BLUE)
  Si OK → step 3

STEP 3 : Canary 25%
  Route 53 : 75% BLUE / 25% GREEN
  Monitoring 30 min
  ↓
  Si OK → step 4

STEP 4 : Canary 50%
  Route 53 : 50% / 50%
  Monitoring 30 min
  ↓
  Si OK → step 5

STEP 5 : Bascule 100%
  Route 53 : 0% BLUE / 100% GREEN
  ↓
  Conservation BLUE 24h (rollback rapide possible)
  ↓
  Destruction BLUE après J+1
```

**Outils** :
- ArgoCD Rollouts (gestion canary native K8s)
- Datadog APM (monitoring canary)
- Flagger (alternative ArgoCD Rollouts)

### D.2 Cadence releases

| Type release | Fréquence | Contenu | Validation |
|--------------|-----------|---------|------------|
| **Major** (v2.0) | 2/an | Features majeures, breaking changes | COPIL + UAT 4 sem |
| **Minor** (v1.x) | Mensuel | Features, améliorations | UAT 1 sem |
| **Patch** (v1.0.x) | Hebdo | Bug fixes, perfs | Tests auto seuls |
| **Hotfix** | Au besoin | Sécurité critique, P1 | Bypass UAT, tests réduits |

**Fenêtres déploiement** :
- Mardi/jeudi 9h-11h (heures bureau ouvrées = équipe disponible si incident)
- **JAMAIS** : vendredi, lundi, veille de weekend, fin/début de mois (clôture)

### D.3 Pipeline CI/CD complet

(Détaillé en Phase 3 §D.4)

**Étapes spécifiques déploiement** :

```yaml
deploy-prod:
  stage: deploy
  environment: production
  rules:
    - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
      when: manual  # Approbation humaine requise
  script:
    # 1. Vérifier signature image
    - cosign verify --key public.pem $IMAGE
    # 2. Backup DB pré-déploiement
    - aws rds create-db-snapshot --db-instance-identifier prod-paraph --db-snapshot-identifier pre-deploy-$CI_COMMIT_SHORT_SHA
    # 3. Helm deploy GREEN
    - helm upgrade --install paraph-green ./charts/parapheur --set image.tag=$CI_COMMIT_TAG --namespace prod
    # 4. Smoke tests
    - npm run test:smoke -- --target green
    # 5. Canary progressif (Flagger)
    - flagger-cli rollout start
```

### D.4 Rollback strategy

**Niveau 1 — Rollback applicatif** (<2 min) :
- Bascule Route 53 BLUE/GREEN → 100% BLUE
- Aucune perte donnée (BD inchangée)

**Niveau 2 — Rollback DB schema** (<10 min) :
- Si migration BDD breaking : utiliser scripts down Flyway
- Restauration snapshot RDS si nécessaire (15 min RTO)

**Niveau 3 — Rollback événements Kafka** (<30 min) :
- Re-consumption topic depuis offset précédent
- Compensation events si actions latérales (SAP postings)

**Niveau 4 — Disaster recovery** (cf. §F.1) :
- Bascule région secondaire (eu-central-1)
- RTO 4h

### D.5 Feature flags

**Outil** : LaunchDarkly (alternative open source : Unleash)

**Patterns** :
- **Kill switch** : désactiver feature instantanément en cas problème (sans rollback déploiement)
- **Progressive rollout** : 1% → 10% → 50% → 100% utilisateurs
- **Targeting** : par filiale, rôle, A/B test

**Usage parapheur** :
- Activation QES par filiale (rollout progressif)
- A/B test UI nouveaux écrans
- Killswitch intégrations critiques (SAP fallback mode)

---

<a name="section-e"></a>
## SECTION E — PLAN DE MIGRATION & COEXISTENCE LEGACY

### E.1 Périmètre legacy actuel

**État existant** :
- ~85% dossiers traités sur **papier** (signature manuscrite scannée)
- ~10% via SharePoint workflow basique
- ~5% via outils ad-hoc (Excel + email)

**Volumétrie historique** : ~100 K dossiers archivés sur 10 ans (boîtes physiques + GED Alfresco).

### E.2 Décision stratégique : pas de migration historique

**Analyse coût/bénéfice migration vs coexistence** :

| Option | Coût | Délai | Bénéfice | Risque |
|--------|------|-------|----------|--------|
| **A. Migration totale** (numériser + import 100K dossiers) | 320 K€ | 12 mois | Recherche unifiée | Corruption données, complexité, retard projet |
| **B. Migration sélective** (5K dossiers actifs) | 65 K€ | 4 mois | Continuité workflows ouverts | Faible (sélection limitée) |
| **C. Coexistence** (legacy reste tel quel) | 45 K€ | 2 mois | Pas de risque migration | Double système 18 mois |

**Recommandation : Option B (sélective) + C (coexistence reste)** :
- Migrer uniquement **dossiers ouverts** au moment du Go Live (~5 K dossiers)
- Archives historiques (95K) → restent dans Alfresco, accessibles en lecture
- Dashboard unifié de recherche (parapheur new + Alfresco legacy)

### E.3 Plan de migration (option B — dossiers actifs)

#### E.3.1 Périmètre

- 5 000 dossiers actifs (workflow non clos au T-Go Live)
- Documents PDF + métadonnées
- Workflows en cours

#### E.3.2 Stratégie migration

**3 vagues sur 6 semaines** :

| Vague | Contenu | Volume | Durée | Stratégie |
|-------|---------|--------|-------|-----------|
| V1 | Dossiers Achats récents (<3 mois) | 1 500 | 2 sem | Re-saisie initiateur (formulaire migration assisté) |
| V2 | Dossiers RH actifs (recrutements) | 800 | 2 sem | Import auto + validation manuelle initiateur |
| V3 | Dossiers Contrats + Capex | 2 700 | 2 sem | Import auto + nettoyage qualité données |

#### E.3.3 Procédure migration (vague V2/V3 — auto)

```
1. EXTRACT : ETL Talend depuis Alfresco + Excel sources
2. TRANSFORM : mapping vers schéma parapheur + enrichissement (référentiels SAP)
3. VALIDATE : règles qualité (95% complétude attributs obligatoires)
4. STAGE : import en environnement staging
5. UAT : initiateur valide ses dossiers (échantillon 10%)
6. LOAD : import production (idempotent, batch)
7. VERIFY : reconciliation comptes (volume V1 = volume migré)
8. CUTOVER : ouverture parapheur pour ces dossiers, fermeture Alfresco écriture
```

#### E.3.4 Rollback migration

- Snapshot DB pré-migration (point-in-time recovery 7j)
- Garde système legacy en lecture 18 mois
- Procédure rollback dossier-par-dossier documentée

### E.4 Coexistence 18 mois

**Pendant la coexistence** :
- Nouveau parapheur : nouveaux dossiers + dossiers migrés
- Alfresco legacy : lecture seule pour archives
- Dashboard unifié (Power BI) consolide les deux pour reporting

**Cutover final (T+18 mois)** :
- Archives Alfresco → eDoc (SAE qualifié)
- Décommissionnement Alfresco
- Réduction OPEX 35K€/an (licences Alfresco)

### E.5 Plan de change management

**Communication & formation** :

| Population | Format | Durée | Total |
|-----------|--------|-------|-------|
| Initiateurs (2400) | E-learning + tutoriel vidéo | 1h | 2 400 h |
| Validateurs (615) | Webinar live + Q&A | 2h | 1 230 h |
| Signataires (85) | Atelier présentiel + démo QES | 4h | 340 h |
| Admin (15) | Formation produit complète | 5j | 75 j |
| Champions par filiale (35) | Train-the-trainers | 2j | 70 j |

**Budget formation** : ~50 K€ (matériel + temps formateurs internes).

**Hypercare post Go Live** :
- 4 semaines support renforcé (équipe dédiée)
- Hotline dédiée parapheur (8h-18h)
- Daily standup équipe support + Dev
- 200+ tickets attendus première semaine, 50/sem stabilisation

---

<a name="section-f"></a>
## SECTION F — DR, CONTINUITÉ, OBSERVABILITÉ

### F.1 Disaster Recovery (DR)

#### F.1.1 Stratégie : Active/Passive cross-region

```
PRIMARY : eu-west-3 (Paris)        SECONDARY : eu-central-1 (Frankfurt)
   ↓                                       ↓
   100% trafic                       0% trafic (warm standby)
   ↓                                       ↓
RDS Multi-AZ ←──── Cross-region replica ──→ RDS read replica
S3 ←──── Cross-region replication ────→ S3
Kafka MSK ←──── MirrorMaker 2 ────→ Kafka MSK
                ↓
         Route 53 health checks
         Failover auto si Paris KO
```

#### F.1.2 Objectifs DR

| Indicateur | Cible | Justification |
|-----------|-------|---------------|
| **RTO** (Recovery Time Objective) | **4 heures** | Coût indispo > coût DR |
| **RPO** (Recovery Point Objective) | **15 minutes** | Async replication tolérable |
| Disponibilité globale | 99,9% (8,76h/an) | SLA contractuel |

#### F.1.3 Procédure DR drill (trimestrielle)

```
T-7j : Communication COPIL drill planifié
T+0 : Simulation panne Paris (suspend ALB)
T+5min : Détection auto Datadog → alerte SOC
T+15min : Décision RSSI/DSI bascule DR
T+30min : Failover Route 53 → eu-central-1
T+45min : Promote RDS read replica → primary
T+1h : Validation services UP (smoke tests)
T+2h : Communication utilisateurs "service opérationnel zone DR"
T+4h : Bascule retour normale (synchronisation BDD)
T+1j : RETEX, mise à jour runbooks
```

**Coût DR** :
- Infra secondary (warm standby 30%) : ~50 K€/an
- Cross-region transfer + replication : ~15 K€/an
- Drill trimestriel + maintenance : ~10 K€/an
- **Total : 75 K€/an** (réajusté vs 65K€ initial)

### F.2 Plan Continuité d'Activité (PCA/PRA)

**Documenté selon ISO 22301** :

| Scénario | Probabilité | Impact | RTO | Mesure |
|----------|------------|--------|-----|--------|
| Indispo région AWS | Très faible | Élevé | 4h | DR cross-region |
| Indispo Universign | Faible | Moyen | 1h | Fallback AdES local |
| Indispo SAP | Moyen | Moyen | 2h | Mode dégradé : workflow continue, postings différés |
| Indispo Okta | Faible | Élevé | 1h | Fallback OAuth local + comptes break-glass |
| Cyberattaque ransomware | Moyen | Très élevé | 24h | Backups immutables + isolation segmentation |
| Indispo internet siège | Moyen | Moyen | N/A | Cloud-based = télétravail OK |

### F.3 Observabilité (3 piliers)

#### F.3.1 Logs (Splunk Cloud)

- **Sources** : tous microservices + ALB + Kong + RDS + EKS + applicatif
- **Volume** : ~50 GB/jour (compressed)
- **Rétention** : 1 an chaud (Splunk) + 6 ans froid (S3 Glacier)
- **Format** : JSON structuré (logback ECS encoder)
- **Champs** : `@timestamp`, `service`, `correlation_id`, `user_id`, `level`, `message`, `trace_id`

**47 use cases SIEM** (cf. Phase 3 §A.6.1)

#### F.3.2 Métriques (Datadog)

**Métriques applicatives** :
- RED Method (Rate, Errors, Duration) par endpoint
- USE Method (Utilization, Saturation, Errors) par ressource
- Business metrics : signatures/h, dossiers en attente, SLA conformity

**Dashboards** :
- **Vue Direction** : KPI globaux, SLA, coûts cloud
- **Vue Ops** : santé infrastructure, alertes
- **Vue Dev** : performances par service, traces
- **Vue Business** : volume dossiers par filiale/type/statut

**Alertes critiques (PagerDuty)** :
- Error rate > 1% sur 5 min → P1
- P95 latency > 2s sur 5 min → P2
- DB connections > 90% → P2
- Disk > 80% → P3
- Échec écriture SAP > 3 consécutifs → P2

#### F.3.3 Traces (Datadog APM + OpenTelemetry)

- **Instrumentation** : auto (Spring Boot starter Datadog)
- **Context propagation** : W3C Trace Context (traceparent header)
- **Sampling** : 10% en prod (100% si error)
- **Service map** : visualisation dépendances inter-services

### F.4 Capacity planning

**Modèle prévisionnel** (Année 1 → Année 5) :

| Métrique | An 1 | An 2 | An 3 | An 4 | An 5 |
|----------|------|------|------|------|------|
| Dossiers/an | 12 K | 15 K | 18 K | 22 K | 25 K |
| Utilisateurs actifs | 3 K | 3,2 K | 3,5 K | 3,8 K | 4 K |
| Taille S3 (TB) | 2 | 4 | 7 | 11 | 16 |
| Taille BDD (GB) | 5 | 12 | 22 | 35 | 50 |
| Coût AWS prod (€/mois) | 8,8K | 10K | 11,5K | 13K | 15K |

**Scaling stratégie** :
- Horizontal scaling auto (HPA) sur EKS
- Vertical scaling RDS si BDD > 30 GB
- Sharding S3 par filiale + année
- Archivage S3 Glacier > 1 an

---

<a name="section-g"></a>
## SECTION G — REGISTRE DES RISQUES ARCHITECTURE (P×I)

| ID | Risque | Catégorie | P | I | P×I | Stratégie | Plan d'action | Owner |
|----|--------|-----------|---|---|-----|-----------|---------------|-------|
| RA-01 | Couplage fort SAP impacte parapheur (cascade indispo) | Intégration | 3 | 4 | 12 | Atténuation | Event-driven Kafka + circuit breaker + fallback async | Archi/DSI |
| RA-02 | Migration Workday 2027 perturbe IAM | Intégration | 4 | 3 | 12 | Atténuation | Connecteur SCIM standard + abstraction provider | Archi |
| RA-03 | Latence Universign dégrade UX signature | Performance | 3 | 3 | 9 | Atténuation | Async + UI feedback + SLA Universign 99,9% | Dev |
| RA-04 | Vendor lock-in AWS | Stratégique | 3 | 3 | 9 | Atténuation | Standards K8s + Terraform multi-cloud + BDD PostgreSQL portable | Archi |
| RA-05 | Coûts cloud explosent (croissance non maîtrisée) | Financier | 3 | 3 | 9 | Atténuation | FinOps : tags + budgets + alertes + revue mensuelle | DAF/DSI |
| RA-06 | Complexité 8 microservices = dette technique | Architecture | 3 | 4 | 12 | Atténuation | Doc archi obligatoire + revue trimestrielle + ADR (Architecture Decision Records) | Archi |
| RA-07 | Échec migration legacy (corruption données) | Migration | 2 | 4 | 8 | Atténuation | UAT exhaustive + rollback procédures + dry runs | Dev/Métier |
| RA-08 | Indispo région AWS Paris >4h | Infrastructure | 1 | 5 | 5 | Atténuation | DR cross-region + drill trimestriel | Ops |
| RA-09 | Saturation Kafka (volumétrie sous-estimée) | Performance | 2 | 4 | 8 | Atténuation | Tests charge + monitoring lag + auto-scale partitions | Dev/Ops |
| RA-10 | Schema Kafka breaking change (backward compat) | Intégration | 3 | 3 | 9 | Atténuation | Schema Registry + versioning + tests contract | Dev |
| RA-11 | Adoption utilisateurs faible (<70% à T+6 mois) | Change Mgmt | 3 | 4 | 12 | Atténuation | Plan formation + champions + UX simple + hypercare | RH/CDM |
| RA-12 | Skill gap équipe (Camunda, Kafka, K8s) | Équipe | 4 | 3 | 12 | Atténuation | Formation + recrutement seniors + accompagnement consultant 6 mois | DSI |
| RA-13 | Délais audit/certification (PASSI, eIDAS) impactent Go Live | Projet | 3 | 4 | 12 | Atténuation | Cabinet sélectionné T-3 mois + budget tampon + audit blanc | RSSI/PMO |
| RA-14 | Échec test perf pic clôture mensuelle | Performance | 2 | 4 | 8 | Atténuation | Tests Gatling spike + auto-scaling + capacité réservée | Dev |
| RA-15 | Conflit gouvernance données (multi-source identités) | Gouvernance | 3 | 3 | 9 | Atténuation | Source-of-truth documenté + Data Stewards filiales | Data Office |

**Risques critiques (P×I ≥ 12)** : RA-01, RA-02, RA-06, RA-11, RA-12, RA-13 → priorisation Sprint 1-3.

---

<a name="section-h"></a>
## SECTION H — PLAN D'IMPLÉMENTATION & GO/NO-GO

### H.1 Planning détaillé 18 semaines

| Sem | Workstream Archi | Workstream Intégration | Workstream Infra/Deploy | Workstream Migration |
|-----|------------------|------------------------|------------------------|----------------------|
| S1-2 | ADR archi finale + revue COPIL | Cartographie SI précisée | Setup AWS accounts + Terraform | Audit legacy + identification dossiers actifs |
| S3-4 | Squelette 8 microservices + CI | Connecteur SAP MDG sandbox | EKS cluster up + Helm charts | ETL Talend conception |
| S5-6 | MS-1 Dossier + MS-2 Workflow MVP | Connecteur Okta SCIM | RDS Multi-AZ + monitoring base | Mapping données legacy → cible |
| S7-8 | MS-3 Signature + integration Universign sandbox | Salesforce + Workday connectors | Kafka MSK + topics + Schema Registry | Premier batch test V1 dossiers achats |
| S9-10 | MS-4 Notification + MS-5 Audit | SAP FI/MM async pipelines | Vault + secrets + HSM CloudHSM | Validation V1 staging |
| S11-12 | MS-6 Référentiels + MS-7 Reporting | eDoc archivage + ServiceNow | DR région secondaire setup | Vague V1 production |
| S13-14 | MS-8 Bus + dashboards monitoring complet | Tests bout-en-bout intégration | Drill DR #1 + ajustements | Vague V2 RH (800 dossiers) |
| S15-16 | Optimisations perf + tuning | Pentest intégrations + fixes | Blue/Green pipeline production | Vague V3 Contrats+Capex |
| S17 | Pre-prod hardening | Validation finale partenaires (Universign audit) | Drill DR #2 + chaos engineering | UAT finale parallel run |
| S18 | Go Live + hypercare | Monitoring intégrations 24/7 | Bascule production complète | Cutover legacy lecture seule |

### H.2 Budget consolidé

#### H.2.1 CAPEX (685 K€)

| Workstream | Détail | Montant |
|-----------|--------|---------|
| **Architecture & Build** | Dev microservices (8) × 8 sem × équipe 6 dev | 285 K€ |
| | Camunda Enterprise licence + setup | 65 K€ |
| | Kong API Gateway Enterprise | 28 K€ |
| | Datadog APM + Logs setup | 20 K€ |
| | Outils dev (LaunchDarkly, ArgoCD, etc.) | 27 K€ |
| **Intégration SI** | Confluent SAP Connector + setup | 45 K€ |
| | Connecteurs Salesforce/Workday/eDoc | 35 K€ |
| | Universign intégration | 22 K€ |
| | API Gateway externalisation | 18 K€ |
| | Tests intégration (CDC, contrats) | 25 K€ |
| | Direct Connect AWS ↔ on-prem | 20 K€ |
| **Déploiement & DR** | Setup EKS + GitOps ArgoCD | 28 K€ |
| | DR cross-region setup | 25 K€ |
| | Tests perf + chaos engineering | 18 K€ |
| | Hypercare 4 semaines équipe renforcée | 24 K€ |
| **Migration legacy** | ETL Talend dev + 3 vagues | 32 K€ |
| | Formation utilisateurs (matériel) | 18 K€ |
| **TOTAL CAPEX** | | **685 K€** |

#### H.2.2 OPEX récurrent (285 K€/an)

| Catégorie | Montant/an |
|-----------|------------|
| AWS infrastructure (prod + dev/staging) | 179 K€ |
| DR cross-region | 75 K€ |
| Camunda Enterprise maintenance | 32 K€ |
| Datadog (logs + APM + monitoring) | 38 K€ |
| Kong Enterprise | 12 K€ |
| Universign (tarifs cf. Phase 3) | 38 K€ |
| LaunchDarkly + outils dev | 15 K€ |
| Support Confluent Kafka | 18 K€ |
| **TOTAL OPEX** | **407 K€/an** (réajusté) |

**Note : OPEX réévalué à 407 K€/an** (vs 285 K€ initial) — incluant DR + outils. À valider COPIL.

### H.3 TCO 5 ans

| Scénario | Year 0 (CAPEX) | Years 1-5 OPEX | TCO 5 ans |
|----------|---------------|----------------|-----------|
| **Cloud microservices** (recommandé) | 685 K€ | 5 × 407 K€ = 2 035 K€ | **2 720 K€** |
| Monolithique on-prem (alternative) | 1 250 K€ | 5 × 480 K€ = 2 400 K€ | **3 650 K€** |
| **Économie microservices** | | | **-930 K€ (-25%)** |

### H.4 Critères GO/NO-GO production

#### H.4.1 GO obligatoires (bloquants)

**Architecture** :
- ☐ 8 microservices déployés et stables
- ☐ Tests intégration 100% verts
- ☐ Performance SLA respectés (P95 < 500ms, dispo 99,9%)
- ☐ Auto-scaling validé (test pic ×3)

**Intégration** :
- ☐ SAP : sync référentiels + écritures FI/MM opérationnelles
- ☐ Okta SSO + MFA fonctionnel pour 100% utilisateurs
- ☐ Universign signatures AdES + QES validées (50 sign tests OK)
- ☐ eDoc archivage : 10 docs test push réussis

**Infrastructure** :
- ☐ DR drill réussi (RTO < 4h validé)
- ☐ Backups testés restauration
- ☐ Monitoring 24/7 opérationnel
- ☐ Runbooks SOC validés

**Migration** :
- ☐ Vagues V1+V2+V3 migrées (5K dossiers)
- ☐ Reconciliation 100% (0 dossier perdu)
- ☐ UAT parallel run validée

**Change management** :
- ☐ Formation 90% utilisateurs réalisée
- ☐ Documentation utilisateur publiée
- ☐ Hotline support opérationnelle

#### H.4.2 NO-GO critères

- 1 microservice instable (uptime <99% en staging)
- Test perf pic KO
- Indispo SAP intégration test
- Pentest critical non corrigé
- DR drill RTO > 6h
- Migration reconciliation < 99%

### H.5 Roadmap post-Go Live (T+6 mois → T+18 mois)

**T+3 mois** :
- Stabilisation hypercare → BAU
- RETEX projet + RETEX utilisateurs
- Premier audit ISO 27001 surveillance

**T+6 mois** :
- Optimisations performance (RUM analyse)
- Roll-out QES filiales DE/CH (initialement FR seulement)
- API B2B partenaires (priorité fournisseurs stratégiques)

**T+12 mois** :
- Migration Workday HR (substitution SAP HR)
- Mobile app native iOS/Android
- IA assistance création dossier (suggestion auto champs)

**T+18 mois** :
- Décommissionnement Alfresco legacy
- Évaluation eIDAS 2.0 + EUDI Wallet
- Extension périmètre (notes de frais ?)

---

<a name="annexes"></a>
## ANNEXES

### Annexe 1 — Architecture Decision Records (ADR)

12 ADR clés documentés :
- ADR-001 : Choix microservices vs monolithique
- ADR-002 : Choix Camunda vs Activiti vs custom
- ADR-003 : Choix Kafka vs RabbitMQ
- ADR-004 : Choix AWS vs Azure vs OVH
- ADR-005 : Choix Universign vs DocuSign
- ADR-006 : Pattern intégration SAP (event-driven)
- ADR-007 : Stratégie migration legacy (coexistence)
- ADR-008 : Stratégie release (Blue/Green Canary)
- ADR-009 : Choix Okta vs ADFS
- ADR-010 : Choix React 18 vs Angular 17
- ADR-011 : Stratégie CI/CD (GitLab + ArgoCD)
- ADR-012 : Choix HSM Thales vs Utimaco

### Annexe 2 — OpenAPI 3.1 specs (résumé)

Specs complètes par microservice (8 fichiers `.yaml`) :
- `paraph-dossier-api.yaml` — 32 endpoints
- `paraph-workflow-api.yaml` — 24 endpoints
- `paraph-signature-api.yaml` — 18 endpoints
- `paraph-notification-api.yaml` — 8 endpoints
- `paraph-audit-api.yaml` — 12 endpoints
- `paraph-referentiel-api.yaml` — 28 endpoints
- `paraph-reporting-api.yaml` — 15 endpoints
- `paraph-integration-api.yaml` — 10 endpoints
- **Total** : 147 endpoints REST documentés

### Annexe 3 — Schémas événements Kafka (Avro)

15 schémas Avro versionnés Schema Registry :
- DossierCreatedV1.avsc, DossierSubmittedV1.avsc, DossierCompletedV1.avsc
- WorkflowStartedV1.avsc, TaskCreatedV1.avsc, TaskCompletedV1.avsc, EscalatedV1.avsc
- SignatureRequestedV1.avsc, SignatureCompletedV1.avsc
- NotificationRequestedV1.avsc
- SapBpChangedV1.avsc, SapPostingRequestedV1.avsc, SapPostingResultV1.avsc
- AuditEventV1.avsc
- HealthCheckV1.avsc

### Annexe 4 — Runbooks opérationnels

10 runbooks formalisés :
- RB-001 : Déploiement production Blue/Green
- RB-002 : Rollback production
- RB-003 : Failover DR cross-region
- RB-004 : Restauration backup BDD
- RB-005 : Réponse incident P1
- RB-006 : Investigation latence dégradée
- RB-007 : Re-consumption Kafka topic
- RB-008 : Rotation clé KMS
- RB-009 : Onboarding nouvelle filiale
- RB-010 : Décommissionnement utilisateur (RGPD)

### Annexe 5 — Glossaire technique

| Acronyme | Définition |
|----------|-----------|
| ADR | Architecture Decision Record |
| ALB | Application Load Balancer |
| APM | Application Performance Monitoring |
| BAPI | Business Application Programming Interface (SAP) |
| BPMN | Business Process Model and Notation |
| CDC | Change Data Capture |
| CMK | Customer Managed Key |
| CQRS | Command Query Responsibility Segregation |
| DMN | Decision Model and Notation |
| EKS | Elastic Kubernetes Service |
| ETL | Extract Transform Load |
| GitOps | Git-based Operations |
| HCM | Human Capital Management |
| HPA | Horizontal Pod Autoscaler |
| IDoc | Intermediate Document (SAP) |
| MDG | Master Data Governance (SAP) |
| MSK | Managed Streaming for Apache Kafka (AWS) |
| OData | Open Data Protocol |
| OPA | Open Policy Agent |
| PCA/PRA | Plan Continuité Activité / Plan Reprise Activité |
| PDB | Pod Disruption Budget |
| PSCo | Prestataire Services Confiance qualifié |
| RDS | Relational Database Service (AWS) |
| RPO | Recovery Point Objective |
| RTO | Recovery Time Objective |
| SAE | Système Archivage Électronique |
| SCIM | System for Cross-domain Identity Management |
| TCO | Total Cost of Ownership |
| tRFC | transactional Remote Function Call (SAP) |
| WAF | Web Application Firewall |

---

## CONCLUSION & SYNTHÈSE GLOBALE 4 PHASES

### Récapitulatif investissement total programme

| Phase | CAPEX | OPEX/an |
|-------|-------|---------|
| Phase 1 — Specs détaillées (US, Archi, UX) | 240 K€ | - |
| Phase 2 — Specs métier (règles, cas usage) | 283 K€ | - |
| Phase 3 — Conformité & Sécurité | 480 K€ | 169 K€ |
| Phase 4 — Architecture & Intégration | 685 K€ | 407 K€ |
| **TOTAL PROGRAMME** | **1 688 K€** | **576 K€/an** |

### TCO 5 ans : **4 568 K€** (CAPEX + 5×OPEX)

### ROI estimé
- Économies dossiers papier : ~85 €/dossier × 12K = **1 020 K€/an**
- Réduction délai validation (8j → 2j) : **gain trésorerie ~280 K€/an**
- Réduction coûts contentieux signature (jurisprudence QES) : **~150 K€/an évités**
- **ROI break-even : ~3,2 ans**

### 17 décisions COPIL à arbitrer (consolidées 4 phases)

**Phase 1** : 3 décisions (microservices, Universign, UX testing)
**Phase 2** : 4 décisions (Camunda DMN, seuils par filiale, SAP MDG, MVP périmètre)
**Phase 3** : 4 décisions (niveau eIDAS, souveraineté, crypto, tests)
**Phase 4** : 5 décisions (intégration SAP, déploiement, legacy, DR, API publique)

### Recommandation finale Direction

**GO programme parapheur digital** :
- Investissement maîtrisé (4,6 M€ TCO 5 ans)
- ROI 3,2 ans (positif, avec gain qualitatif additionnel)
- Risques identifiés et plans d'action chiffrés
- Architecture évolutive, conforme normes
- Expérience utilisateur priorisée
- Conformité totale (RGPD + eIDAS + ISO 27001)

**Conditions de succès** :
1. Sponsorship Direction Générale visible
2. Équipe pluridisciplinaire dédiée 18 mois
3. Budget protégé (pas de coupes en cours)
4. Change management musclé
5. Hypercare 4 semaines minimum

---

**Validation requise** :

| Rôle | Nom | Date | Signature |
|------|-----|------|-----------|
| Chef de Programme | | | |
| Architecte d'Entreprise | | | |
| DSI | | | |
| RSSI | | | |
| DPO | | | |
| Direction Juridique | | | |
| DAF | | | |
| Directeur Général | | | |

**Fin du document Phase 4 — v1.0**
**Fin du programme de spécifications complet (Phases 1+2+3+4)**
