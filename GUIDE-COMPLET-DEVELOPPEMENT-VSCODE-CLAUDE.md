# Guide de développement — Parapheur Digital
## Procédure complète : VS Code + Claude → Application professionnelle en production
### Maroc | Solo Developer | Node.js / React / PostgreSQL
**Version 1.0 — Avril 2026**

---

## TABLE DES MATIÈRES

1. [Vue d'ensemble et principes directeurs](#1-vue-densemble-et-principes-directeurs)
2. [Prérequis et configuration de l'environnement](#2-prérequis-et-configuration-de-lenvironnement)
3. [Initialisation du monorepo](#3-initialisation-du-monorepo)
4. [Base de données PostgreSQL](#4-base-de-données-postgresql)
5. [API Gateway et fondations backend](#5-api-gateway-et-fondations-backend)
6. [Microservices backend (MS-1 à MS-6)](#6-microservices-backend-ms-1-à-ms-6)
7. [Frontend React (interface utilisateur)](#7-frontend-react-interface-utilisateur)
8. [Intégrations externes (e-signature, GED, AD)](#8-intégrations-externes-e-signature-ged-ad)
9. [Tests et qualité (TDD/BDD)](#9-tests-et-qualité-tddbdd)
10. [Sécurité et conformité (DNSSI/ISO 27001/Loi 43-20)](#10-sécurité-et-conformité-dnssiiso-27001loi-43-20)
11. [CI/CD et déploiement](#11-cicd-et-déploiement)
12. [Opérations et monitoring](#12-opérations-et-monitoring)
13. [Workflow Claude — Comment travailler avec l'IA](#13-workflow-claude--comment-travailler-avec-lia)
14. [Checklist de livraison](#14-checklist-de-livraison)

---

## 1. VUE D'ENSEMBLE ET PRINCIPES DIRECTEURS

### 1.1 Ce que vous allez construire

Une application web fullstack de gestion de parapheur digital pour une banque marocaine cotée :
- **12 000 dossiers/an** (50/jour ouvré)
- **Workflow multi-niveaux** (N1 à N6, comités, délégations)
- **Signature électronique** (SES/SEA/SEQ — Loi 43-20)
- **Interface bilingue** FR/AR avec RTL (arabe droite-à-gauche)
- **Conformité** : Loi 09-08, Loi 43-20, Loi 05-20, BAM, DGSSI, AMMC

### 1.2 Stack technique définitif

```
BACKEND (Node.js 20 LTS)
├── Runtime       : Node.js 20 LTS + TypeScript 5.4
├── Framework API : Fastify 4.x (vs Express: 3x plus rapide)
├── ORM           : Prisma 5.x (type-safe, migrations, PostgreSQL)
├── Workflow      : XState 5 (state machines — Loi 43-20 compliance)
├── Queue         : BullMQ + Redis 7 (jobs asynchrones)
├── Validation    : Zod 3.x (schemas partagés front/back)
└── Auth          : LDAP (Active Directory banque) + TOTP (2FA)

FRONTEND (React 18)
├── Build         : Vite 5.x (< 100ms HMR)
├── UI            : shadcn/ui + Tailwind CSS 3.x
├── State         : TanStack Query v5 (server state) + Zustand (client state)
├── Forms         : React Hook Form + Zod
├── i18n          : i18next + react-i18next + Tailwind RTL
├── PWA           : Vite PWA plugin (offline, push notifications)
└── Tests         : Vitest + Testing Library + Playwright (e2e)

DONNÉES
├── Primaire      : PostgreSQL 16 (JSONB, full-text FR+AR, row-level security)
├── Cache/Queue   : Redis 7 (session, cache taux BAM, BullMQ)
└── Fichiers      : GED interne (adapter REST)

INFRASTRUCTURE
├── Conteneurs    : Docker + Docker Compose (dev) → production (DC Maroc)
├── CI/CD         : GitHub Actions (lint → test → build → deploy)
├── Observabilité : Prometheus + Grafana + Loki (logs structurés JSON)
└── Secrets       : Vault ou env secrets chiffrés
```

### 1.3 Principes de développement avec Claude

**Règle #1 — Un contexte par conversation :** Ouvrir une nouvelle conversation Claude pour chaque microservice ou composant majeur. Ne pas mélanger les sujets.

**Règle #2 — Spec avant code :** Toujours fournir à Claude le fichier de spec correspondant avant de demander du code. Claude produit du code professionnel si le contexte est riche.

**Règle #3 — TDD systématique :** Demander à Claude de générer les tests *avant* l'implémentation. `test → fail → implement → pass → refactor`.

**Règle #4 — Review chaque PR :** Même en solo, créer une PR par feature. Claude peut faire la code review.

**Règle #5 — Commits atomiques :** Un commit = une fonctionnalité précise. Message format : `feat(ms-dossier): add dossier creation endpoint with Zod validation`.

### 1.4 Architecture des dossiers (vue globale)

```
parapheur-digital/              ← racine monorepo
├── apps/
│   ├── api-gateway/            ← Fastify API Gateway (port 3000)
│   ├── ms-dossier/             ← Microservice Dossier (port 3001)
│   ├── ms-workflow/            ← Microservice Workflow XState (port 3002)
│   ├── ms-notification/        ← Microservice Notifications (port 3003)
│   ├── ms-audit/               ← Microservice Audit (port 3004)
│   ├── ms-esig-adapter/        ← Adapter e-signature interne (port 3005)
│   ├── ms-ged-adapter/         ← Adapter GED interne (port 3006)
│   └── web/                    ← Frontend React (port 5173)
├── packages/
│   ├── shared-types/           ← Types TypeScript partagés
│   ├── shared-schemas/         ← Schemas Zod partagés front/back
│   ├── shared-utils/           ← Utilitaires communs
│   ├── shared-db/              ← Client Prisma + migrations
│   └── shared-config/          ← Config ESLint, Prettier, TS
├── infra/
│   ├── docker/                 ← Dockerfiles
│   ├── compose/                ← docker-compose files
│   └── scripts/                ← Scripts CI/CD, migrations
├── docs/                       ← Specs + ADR + Runbooks
├── .github/
│   └── workflows/              ← GitHub Actions CI/CD
├── package.json                ← Workspace root (pnpm)
├── pnpm-workspace.yaml
├── turbo.json                  ← Turborepo pipelines
└── docker-compose.yml          ← Dev environment
```

---

## 2. PRÉREQUIS ET CONFIGURATION DE L'ENVIRONNEMENT

### 2.1 Installation des outils système

**Ouvrir le terminal et exécuter dans l'ordre :**

```bash
# 1. Homebrew (macOS) ou apt (Linux)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 2. Node.js via nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc  # ou ~/.zshrc
nvm install 20
nvm use 20
nvm alias default 20
node --version  # doit afficher v20.x.x

# 3. pnpm (gestionnaire de paquets monorepo)
npm install -g pnpm@9
pnpm --version  # doit afficher 9.x.x

# 4. PostgreSQL 16
brew install postgresql@16
brew services start postgresql@16
psql --version  # doit afficher psql (PostgreSQL) 16.x

# 5. Redis 7
brew install redis
brew services start redis
redis-cli ping  # doit répondre PONG

# 6. Docker Desktop
brew install --cask docker
# Ouvrir Docker Desktop et attendre le démarrage

# 7. Git (si pas déjà installé)
brew install git
git --version

# 8. GitHub CLI
brew install gh
gh auth login  # suivre les instructions

# 9. Outils dev globaux
npm install -g tsx                  # exécuter TypeScript directement
npm install -g turbo                # Turborepo (monorepo)
npm install -g @prisma/cli          # Prisma CLI
```

### 2.2 Configuration VS Code

**Extensions VS Code OBLIGATOIRES** (Ctrl+P → `ext install <id>`) :

```
# TypeScript & Node.js
dbaeumer.vscode-eslint
esbenp.prettier-vscode
bradlc.vscode-tailwindcss
ms-vscode.vscode-typescript-next

# Prisma (ORM PostgreSQL)
Prisma.prisma

# Tests
vitest.explorer

# Git & GitHub
eamodio.gitlens
github.vscode-pull-request-github
github.copilot  (si disponible)

# Base de données
cweijan.vscode-postgresql-client2

# REST API
humao.rest-client

# Containers
ms-azuretools.vscode-docker
ms-vscode-remote.remote-containers

# Qualité
streetsidesoftware.code-spell-checker
christian-kohler.path-intellisense

# Diagrammes
hediet.vscode-drawio
```

**Configuration VS Code** — Créer `.vscode/settings.json` à la racine du projet :

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,
  "tailwindCSS.experimental.classRegex": [
    ["cva\\(([^)]*)\\)", "[\"'`]([^\"'`]*).*?[\"'`]"],
    ["cx\\(([^)]*)\\)", "(?:'|\"|`)([^']*)(?:'|\"|`)"]
  ],
  "files.associations": {
    "*.css": "tailwindcss"
  },
  "prisma.showPrismaDataPlatformNotification": false,
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

**Créer `.vscode/extensions.json`** :

```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "bradlc.vscode-tailwindcss",
    "Prisma.prisma",
    "vitest.explorer",
    "eamodio.gitlens",
    "humao.rest-client",
    "ms-azuretools.vscode-docker",
    "streetsidesoftware.code-spell-checker"
  ]
}
```

### 2.3 Configuration Git

```bash
# Identité Git
git config --global user.name "Votre Nom"
git config --global user.email "votre@email.com"

# Branche par défaut : main
git config --global init.defaultBranch main

# Éditeur
git config --global core.editor "code --wait"

# Merge strategy
git config --global pull.rebase false
```

### 2.4 Variables d'environnement

**Créer le fichier `.env.example` à la racine :** (sera copié en `.env.local` pour chaque dev)

```bash
# === BASE DE DONNÉES ===
DATABASE_URL="postgresql://parapheur:password@localhost:5432/parapheur_dev"
DATABASE_SHADOW_URL="postgresql://parapheur:password@localhost:5432/parapheur_shadow"

# === REDIS ===
REDIS_URL="redis://localhost:6379"

# === AUTHENTIFICATION ===
LDAP_URL="ldap://ad.banque.local:389"
LDAP_BASE_DN="dc=banque,dc=local"
LDAP_BIND_DN="cn=service-parapheur,ou=services,dc=banque,dc=local"
LDAP_BIND_PASSWORD="changeme"

JWT_SECRET="changeme-minimum-32-chars-en-production"
JWT_EXPIRY="8h"
SESSION_SECRET="changeme-minimum-32-chars-en-production"
TOTP_ISSUER="Parapheur Digital - Banque"

# === API E-SIGNATURE (plateforme interne) ===
ESIG_BASE_URL="https://esig.banque.local/api/v1"
ESIG_API_KEY="changeme"
ESIG_CALLBACK_URL="https://parapheur.banque.local/api/v1/callbacks/esig"
ESIG_TIMEOUT_MS=30000

# === GED (plateforme GED interne) ===
GED_BASE_URL="https://ged.banque.local/api/v2"
GED_API_KEY="changeme"
GED_DEFAULT_FOLDER_ID="parapheur-digital"
GED_RETENTION_YEARS=10

# === API CORE BANKING ===
CORE_BANKING_URL="https://corebanking.banque.local/api"
CORE_BANKING_API_KEY="changeme"

# === NOTIFICATIONS ===
SMTP_HOST="smtp.banque.local"
SMTP_PORT=587
SMTP_USER="parapheur@banque.ma"
SMTP_PASSWORD="changeme"
SMTP_FROM="Parapheur Digital <parapheur@banque.ma>"

SMS_PROVIDER_URL="https://sms.provider.ma/api"
SMS_API_KEY="changeme"
SMS_FROM="BANQUE"

# === TAUX DE CHANGE (BAM) ===
BAM_API_URL="https://www.bkam.ma/api/exchange-rates"
BAM_API_CACHE_TTL=3600

# === MONITORING ===
LOG_LEVEL="info"
PROMETHEUS_PORT=9090

# === ENVIRONNEMENT ===
NODE_ENV="development"
PORT=3000
FRONTEND_URL="http://localhost:5173"
```

---

## 3. INITIALISATION DU MONOREPO

### 3.1 Créer la structure de base

```bash
# Cloner le repo existant (ou créer nouveau)
git clone https://github.com/tkassi/parapheur-digital.git
cd parapheur-digital

# Créer la structure des dossiers
mkdir -p apps/{api-gateway,ms-dossier,ms-workflow,ms-notification,ms-audit,ms-esig-adapter,ms-ged-adapter,web}
mkdir -p packages/{shared-types,shared-schemas,shared-utils,shared-db,shared-config}
mkdir -p infra/{docker,compose,scripts}
mkdir -p docs/{adr,runbooks,api}
mkdir -p .github/workflows
```

### 3.2 Fichiers de configuration monorepo racine

**`pnpm-workspace.yaml`** :
```yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

**`package.json`** racine :
```json
{
  "name": "parapheur-digital",
  "private": true,
  "scripts": {
    "build": "turbo build",
    "dev": "turbo dev",
    "test": "turbo test",
    "lint": "turbo lint",
    "format": "prettier --write \"**/*.{ts,tsx,json,md}\"",
    "db:generate": "pnpm --filter @parapheur/db prisma generate",
    "db:migrate": "pnpm --filter @parapheur/db prisma migrate dev",
    "db:seed": "pnpm --filter @parapheur/db prisma db seed",
    "db:studio": "pnpm --filter @parapheur/db prisma studio",
    "docker:up": "docker-compose -f docker-compose.yml up -d",
    "docker:down": "docker-compose -f docker-compose.yml down",
    "docker:logs": "docker-compose logs -f"
  },
  "devDependencies": {
    "turbo": "^2.0.0",
    "prettier": "^3.2.5",
    "typescript": "^5.4.5"
  },
  "engines": {
    "node": ">=20.0.0",
    "pnpm": ">=9.0.0"
  }
}
```

**`turbo.json`** :
```json
{
  "$schema": "https://turbo.build/schema.json",
  "globalEnv": ["NODE_ENV", "DATABASE_URL", "REDIS_URL"],
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": ["coverage/**"]
    },
    "lint": {
      "outputs": []
    }
  }
}
```

**`.prettierrc`** :
```json
{
  "semi": false,
  "singleQuote": true,
  "printWidth": 100,
  "trailingComma": "es5",
  "tabWidth": 2
}
```

### 3.3 Configuration TypeScript partagée

**`packages/shared-config/tsconfig.base.json`** :
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022"],
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  }
}
```

### 3.4 Docker Compose pour développement

**`docker-compose.yml`** à la racine :
```yaml
version: '3.9'

services:
  postgres:
    image: postgres:16-alpine
    container_name: parapheur_postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: parapheur
      POSTGRES_PASSWORD: password
      POSTGRES_DB: parapheur_dev
      POSTGRES_INITDB_ARGS: "--encoding=UTF8 --locale=fr_FR.UTF-8"
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./infra/scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U parapheur -d parapheur_dev"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: parapheur_redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  maildev:
    image: maildev/maildev
    container_name: parapheur_maildev
    ports:
      - "1080:1080"   # UI web
      - "1025:1025"   # SMTP
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

**Démarrer l'environnement :**
```bash
docker-compose up -d
# Vérifier que tout tourne
docker-compose ps
# PostgreSQL accessible sur localhost:5432
# Redis accessible sur localhost:6379
# Maildev (UI) accessible sur http://localhost:1080
```

### 3.5 ESLint configuration partagée

**`packages/shared-config/eslint-config/index.js`** :
```js
module.exports = {
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/strict-type-checked',
    'plugin:@typescript-eslint/stylistic-type-checked',
    'prettier',
  ],
  plugins: ['@typescript-eslint', 'import'],
  rules: {
    '@typescript-eslint/no-explicit-any': 'error',
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    'import/order': ['error', {
      'groups': ['builtin', 'external', 'internal', 'parent', 'sibling'],
      'newlines-between': 'always',
      'alphabetize': { order: 'asc' }
    }],
    'no-console': ['warn', { allow: ['warn', 'error'] }],
  },
}
```

---

## 4. BASE DE DONNÉES POSTGRESQL

### 4.1 Initialisation du package `shared-db`

```bash
cd packages/shared-db
pnpm init
pnpm add prisma @prisma/client
pnpm add -D typescript @types/node
npx prisma init
```

### 4.2 Schéma Prisma principal

**`packages/shared-db/prisma/schema.prisma`** :

```prisma
generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["fullTextSearch", "fullTextIndex"]
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ============================================================
// AUTHENTIFICATION ET UTILISATEURS
// ============================================================

model User {
  id          String   @id @default(uuid())
  employeeId  String   @unique @map("employee_id")
  email       String   @unique
  fullNameFr  String   @map("full_name_fr")
  fullNameAr  String?  @map("full_name_ar")
  title       String?
  department  String?
  level       AuthLevel
  isActive    Boolean  @default(true) @map("is_active")
  lastLoginAt DateTime? @map("last_login_at")
  createdAt   DateTime @default(now()) @map("created_at")
  updatedAt   DateTime @updatedAt @map("updated_at")

  // Relations
  dossierCreated    Dossier[]         @relation("DossierCreator")
  dossierValidation ValidationTask[]
  delegationsGiven  Delegation[]      @relation("DelegationGrantor")
  delegationReceived Delegation[]     @relation("DelegationGrantee")
  auditLogs         AuditLog[]

  @@map("users")
}

enum AuthLevel {
  N1_DG
  N2_DGA
  N3_DIR_CENTRALE
  N4_DIR_DEPT
  N5_RESP_AGENCE
  N6_CHARGE
  AUDITEUR
  RSSI
  ADMIN
}

model Delegation {
  id          String   @id @default(uuid())
  grantorId   String   @map("grantor_id")
  granteeId   String   @map("grantee_id")
  reason      String?
  startDate   DateTime @map("start_date")
  endDate     DateTime @map("end_date")
  isActive    Boolean  @default(false) @map("is_active")
  activatedAt DateTime? @map("activated_at")
  createdAt   DateTime @default(now()) @map("created_at")

  grantor     User     @relation("DelegationGrantor", fields: [grantorId], references: [id])
  grantee     User     @relation("DelegationGrantee", fields: [granteeId], references: [id])

  @@map("delegations")
}

// ============================================================
// DOSSIERS
// ============================================================

model Dossier {
  id              String        @id @default(uuid())
  reference       String        @unique
  title           String
  titleAr         String?       @map("title_ar")
  type            DossierType
  status          DossierStatus @default(BROUILLON)
  montant         Decimal       @db.Decimal(20, 2)
  devise          String        @default("MAD")
  montantMad      Decimal       @db.Decimal(20, 2) @map("montant_mad")
  tauxChange      Decimal?      @db.Decimal(10, 6) @map("taux_change")
  contrepartieId  String?       @map("contrepartie_id")
  contrepartieNom String?       @map("contrepartie_nom")
  objet           String
  objetAr         String?       @map("objet_ar")
  urgence         Boolean       @default(false)
  derogation      Boolean       @default(false)
  metadata        Json          @default("{}")
  creatorId       String        @map("creator_id")
  submittedAt     DateTime?     @map("submitted_at")
  completedAt     DateTime?     @map("completed_at")
  dueDate         DateTime?     @map("due_date")
  createdAt       DateTime      @default(now()) @map("created_at")
  updatedAt       DateTime      @updatedAt @map("updated_at")

  // Full-text search
  searchVector    Unsupported("tsvector")? @map("search_vector")

  // Relations
  creator         User              @relation("DossierCreator", fields: [creatorId], references: [id])
  documents       Document[]
  workflowInstance WorkflowInstance?
  auditLogs       AuditLog[]

  @@index([status])
  @@index([type])
  @@index([creatorId])
  @@index([createdAt])
  @@map("dossiers")
}

enum DossierType {
  ENGAGEMENT_CREDIT
  CONTRAT_COMMERCIAL
  DECISION_RH
  ACHAT_INVESTISSEMENT
  ACTE_REGLEMENTAIRE
  AUTRE
}

enum DossierStatus {
  BROUILLON
  SOUMIS
  EN_COURS
  EN_ATTENTE_INFO
  APPROUVE
  REJETE
  ANNULE
  ARCHIVE
}

// ============================================================
// DOCUMENTS
// ============================================================

model Document {
  id           String   @id @default(uuid())
  dossierId    String   @map("dossier_id")
  nom          String
  type         String   // MIME type
  taille       Int
  gedDocumentId String? @map("ged_document_id")
  gedUrl       String?  @map("ged_url")
  hash         String   // SHA-256 pour intégrité
  isRequired   Boolean  @default(false) @map("is_required")
  uploadedAt   DateTime @default(now()) @map("uploaded_at")
  uploadedBy   String   @map("uploaded_by")

  dossier      Dossier  @relation(fields: [dossierId], references: [id])

  @@map("documents")
}

// ============================================================
// WORKFLOW
// ============================================================

model WorkflowInstance {
  id              String   @id @default(uuid())
  dossierId       String   @unique @map("dossier_id")
  currentState    String   @map("current_state")
  machineSnapshot Json     @map("machine_snapshot")  // état XState sérialisé
  startedAt       DateTime @default(now()) @map("started_at")
  completedAt     DateTime? @map("completed_at")
  updatedAt       DateTime @updatedAt @map("updated_at")

  dossier         Dossier          @relation(fields: [dossierId], references: [id])
  tasks           ValidationTask[]
  transitions     WorkflowTransition[]

  @@map("workflow_instances")
}

model ValidationTask {
  id                 String     @id @default(uuid())
  workflowInstanceId String     @map("workflow_instance_id")
  assigneeId         String     @map("assignee_id")
  step               Int
  level              AuthLevel
  status             TaskStatus @default(PENDING)
  signatureType      SignatureType @map("signature_type")
  signatureRequestId String?    @map("signature_request_id")
  decision           TaskDecision? @map("decision")
  comment            String?
  slaDeadline        DateTime   @map("sla_deadline")
  completedAt        DateTime?  @map("completed_at")
  createdAt          DateTime   @default(now()) @map("created_at")
  updatedAt          DateTime   @updatedAt @map("updated_at")

  workflowInstance   WorkflowInstance @relation(fields: [workflowInstanceId], references: [id])
  assignee           User             @relation(fields: [assigneeId], references: [id])

  @@map("validation_tasks")
}

model WorkflowTransition {
  id                 String   @id @default(uuid())
  workflowInstanceId String   @map("workflow_instance_id")
  fromState          String   @map("from_state")
  toState            String   @map("to_state")
  event              String
  actorId            String?  @map("actor_id")
  comment            String?
  timestamp          DateTime @default(now())

  workflowInstance   WorkflowInstance @relation(fields: [workflowInstanceId], references: [id])

  @@index([workflowInstanceId])
  @@map("workflow_transitions")
}

enum TaskStatus {
  PENDING
  IN_PROGRESS
  COMPLETED
  CANCELLED
  DELEGATED
  ESCALATED
}

enum TaskDecision {
  APPROVE
  REJECT
  REQUEST_INFO
  ESCALATE
}

enum SignatureType {
  SES
  SEA
  SEQ
  NONE
}

// ============================================================
// AUDIT (append-only, jamais de UPDATE/DELETE)
// ============================================================

model AuditLog {
  id         BigInt   @id @default(autoincrement())
  entityType String   @map("entity_type")
  entityId   String   @map("entity_id")
  action     String
  actorId    String?  @map("actor_id")
  actorEmail String?  @map("actor_email")
  before     Json?
  after      Json?
  ipAddress  String?  @map("ip_address")
  userAgent  String?  @map("user_agent")
  hash       String   // SHA-256(prev_hash + payload)
  prevHash   String?  @map("prev_hash")
  timestamp  DateTime @default(now())
  dossierId  String?  @map("dossier_id")

  actor      User?    @relation(fields: [actorId], references: [id])
  dossier    Dossier? @relation(fields: [dossierId], references: [id])

  @@index([entityType, entityId])
  @@index([timestamp])
  @@index([actorId])
  @@map("audit_logs")
}
```

### 4.3 Migrations initiales

```bash
# Créer la première migration
cd packages/shared-db
npx prisma migrate dev --name init

# Créer les indexes de recherche full-text (migration SQL manuelle)
# Créer packages/shared-db/prisma/migrations/20260426_fts/migration.sql
```

**`prisma/migrations/20260426_fts/migration.sql`** :
```sql
-- Full-text search: colonne tsvector calculée automatiquement
ALTER TABLE dossiers ADD COLUMN IF NOT EXISTS search_vector tsvector
  GENERATED ALWAYS AS (
    setweight(to_tsvector('french', coalesce(title, '')), 'A') ||
    setweight(to_tsvector('french', coalesce(objet, '')), 'B') ||
    setweight(to_tsvector('simple', coalesce(reference, '')), 'A') ||
    setweight(to_tsvector('simple', coalesce(contrepartie_nom, '')), 'B')
  ) STORED;

CREATE INDEX IF NOT EXISTS dossiers_search_vector_idx ON dossiers USING GIN(search_vector);

-- Audit log: désactiver les mises à jour (append-only)
CREATE RULE audit_no_update AS ON UPDATE TO audit_logs DO INSTEAD NOTHING;
CREATE RULE audit_no_delete AS ON DELETE TO audit_logs DO INSTEAD NOTHING;

-- Index de performance sur audit_logs
CREATE INDEX IF NOT EXISTS audit_logs_dossier_idx ON audit_logs(dossier_id) WHERE dossier_id IS NOT NULL;
CREATE INDEX IF NOT EXISTS audit_logs_actor_time_idx ON audit_logs(actor_id, timestamp DESC);
```

### 4.4 Seed de données de test

**`packages/shared-db/prisma/seed.ts`** :
```typescript
import { PrismaClient, AuthLevel, DossierType, DossierStatus } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  console.log('Seeding database...')

  // Créer les utilisateurs de test (personas de la spec PHASE1)
  const users = await Promise.all([
    prisma.user.upsert({
      where: { employeeId: 'EMP001' },
      update: {},
      create: {
        employeeId: 'EMP001',
        email: 'khalid.alami@banque.ma',
        fullNameFr: 'Khalid Alami',
        fullNameAr: 'خالد العلمي',
        title: 'Chargé de Clientèle Entreprises',
        department: 'Banque Commerciale',
        level: AuthLevel.N6_CHARGE,
      },
    }),
    prisma.user.upsert({
      where: { employeeId: 'EMP002' },
      update: {},
      create: {
        employeeId: 'EMP002',
        email: 'aicha.benali@banque.ma',
        fullNameFr: 'Aïcha Benali',
        fullNameAr: 'عائشة بن علي',
        title: 'Directrice Crédit Entreprises',
        department: 'Crédit',
        level: AuthLevel.N4_DIR_DEPT,
      },
    }),
    prisma.user.upsert({
      where: { employeeId: 'EMP003' },
      update: {},
      create: {
        employeeId: 'EMP003',
        email: 'mohammed.tazi@banque.ma',
        fullNameFr: 'Mohammed Tazi',
        fullNameAr: 'محمد الطازي',
        title: 'Directeur Général Adjoint Banque Commerciale',
        department: 'Direction Générale',
        level: AuthLevel.N2_DGA,
      },
    }),
    prisma.user.upsert({
      where: { employeeId: 'EMP099' },
      update: {},
      create: {
        employeeId: 'EMP099',
        email: 'yassir.mansouri@banque.ma',
        fullNameFr: 'Yassir Mansouri',
        fullNameAr: 'ياسر المنصوري',
        title: 'Auditeur Interne Senior',
        department: 'Audit Interne',
        level: AuthLevel.AUDITEUR,
      },
    }),
  ])

  console.log(`Created ${users.length} test users`)

  // Créer un dossier de test
  const dossier = await prisma.dossier.upsert({
    where: { reference: 'DOS-2026-0001' },
    update: {},
    create: {
      reference: 'DOS-2026-0001',
      title: 'Engagement crédit - Société TEST SA',
      type: DossierType.ENGAGEMENT_CREDIT,
      status: DossierStatus.BROUILLON,
      montant: 800000,
      devise: 'MAD',
      montantMad: 800000,
      contrepartieNom: 'Société TEST SA',
      objet: 'Ligne de crédit d\'exploitation - renouvellement annuel',
      creatorId: users[0]!.id,
    },
  })

  console.log(`Created test dossier: ${dossier.reference}`)
  console.log('Seeding complete!')
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect())
```

---

## 5. API GATEWAY ET FONDATIONS BACKEND

### 5.1 Initialisation de l'API Gateway

```bash
cd apps/api-gateway
pnpm init
pnpm add fastify @fastify/cors @fastify/helmet @fastify/rate-limit @fastify/swagger @fastify/auth
pnpm add @fastify/cookie @fastify/session @fastify/jwt
pnpm add zod fastify-zod-openapi
pnpm add pino pino-pretty  # logging structuré
pnpm add ldapjs @types/ldapjs  # Active Directory
pnpm add speakeasy qrcode  # TOTP 2FA
pnpm add -D typescript @types/node tsx vitest
```

### 5.2 Structure de l'API Gateway

```
apps/api-gateway/src/
├── index.ts              ← point d'entrée
├── app.ts                ← configuration Fastify
├── config.ts             ← variables d'environnement validées (Zod)
├── plugins/
│   ├── cors.ts           ← CORS configuré pour les domaines banque
│   ├── auth.ts           ← JWT + LDAP auth plugin
│   ├── rateLimit.ts      ← rate limiting par endpoint
│   └── swagger.ts        ← documentation OpenAPI auto-générée
├── routes/
│   ├── auth/
│   │   ├── login.ts      ← POST /auth/login (LDAP + JWT)
│   │   ├── totp.ts       ← POST /auth/totp/verify
│   │   ├── logout.ts     ← POST /auth/logout
│   │   └── refresh.ts    ← POST /auth/refresh
│   ├── health.ts         ← GET /health
│   └── proxy/            ← proxy vers microservices
├── middleware/
│   ├── authenticate.ts   ← vérification JWT
│   ├── authorize.ts      ← vérification rôles/niveaux
│   └── auditLogger.ts    ← log chaque requête
└── types/
    └── fastify.d.ts      ← types augmentés Fastify
```

### 5.3 Point d'entrée principal

**`apps/api-gateway/src/app.ts`** :
```typescript
import Fastify from 'fastify'
import cors from '@fastify/cors'
import helmet from '@fastify/helmet'
import rateLimit from '@fastify/rate-limit'
import jwt from '@fastify/jwt'
import { config } from './config.js'

export function buildApp() {
  const app = Fastify({
    logger: {
      level: config.LOG_LEVEL,
      transport: config.NODE_ENV === 'development'
        ? { target: 'pino-pretty', options: { colorize: true } }
        : undefined,
    },
    trustProxy: true,
  })

  // Security headers (DNSSI compliance)
  app.register(helmet, {
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", 'data:'],
        connectSrc: ["'self'"],
        frameSrc: ["'none'"],
      },
    },
    hsts: { maxAge: 31536000, includeSubDomains: true },
  })

  // CORS: uniquement domaines banque
  app.register(cors, {
    origin: [config.FRONTEND_URL, 'https://parapheur.banque.local'],
    methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
    credentials: true,
  })

  // Rate limiting (protection brute force)
  app.register(rateLimit, {
    max: 100,
    timeWindow: '1 minute',
    // Rate limit plus strict sur /auth
    errorResponseBuilder: (req, context) => ({
      statusCode: 429,
      error: 'Too Many Requests',
      message: `Trop de requêtes. Réessayez dans ${context.after}`,
    }),
  })

  // JWT
  app.register(jwt, {
    secret: config.JWT_SECRET,
    sign: { expiresIn: config.JWT_EXPIRY },
  })

  // Routes
  app.register(import('./routes/health.js'))
  app.register(import('./routes/auth/login.js'), { prefix: '/api/v1/auth' })
  app.register(import('./routes/auth/totp.js'), { prefix: '/api/v1/auth' })

  // Proxy vers microservices (en production, utiliser un reverse proxy)
  app.register(import('./routes/proxy/dossiers.js'), { prefix: '/api/v1' })
  app.register(import('./routes/proxy/workflow.js'), { prefix: '/api/v1' })
  app.register(import('./routes/proxy/audit.js'), { prefix: '/api/v1' })

  return app
}
```

### 5.4 Route d'authentification LDAP

**`apps/api-gateway/src/routes/auth/login.ts`** :
```typescript
import { FastifyInstance } from 'fastify'
import ldap from 'ldapjs'
import { z } from 'zod'
import { config } from '../../config.js'

const loginSchema = z.object({
  username: z.string().min(3).max(50),
  password: z.string().min(8).max(200),
})

export async function loginRoutes(app: FastifyInstance) {
  app.post('/login', {
    config: { rateLimit: { max: 5, timeWindow: '5 minutes' } }, // brute force protection
    schema: {
      description: 'Authentification LDAP Active Directory',
      body: loginSchema,
      response: {
        200: {
          type: 'object',
          properties: {
            token: { type: 'string' },
            requireTotp: { type: 'boolean' },
            user: {
              type: 'object',
              properties: {
                id: { type: 'string' },
                email: { type: 'string' },
                fullNameFr: { type: 'string' },
                level: { type: 'string' },
              },
            },
          },
        },
      },
    },
    async handler(request, reply) {
      const { username, password } = loginSchema.parse(request.body)

      // Authentification LDAP
      const user = await authenticateWithLdap(username, password)

      if (!user) {
        return reply.code(401).send({
          statusCode: 401,
          error: 'Unauthorized',
          message: 'Identifiant ou mot de passe incorrect',
        })
      }

      // Vérifier si TOTP est requis (selon niveau N2+)
      const requireTotp = user.level !== 'N6_CHARGE' && user.level !== 'N5_RESP_AGENCE'

      // Générer JWT temporaire (pour finaliser avec TOTP si requis)
      const token = app.jwt.sign(
        {
          sub: user.id,
          email: user.email,
          level: user.level,
          totpVerified: !requireTotp,
        },
        { expiresIn: requireTotp ? '5m' : config.JWT_EXPIRY } // 5min si TOTP requis
      )

      // Audit log
      app.log.info({ action: 'auth.login', userId: user.id, requireTotp })

      return { token, requireTotp, user }
    },
  })
}

async function authenticateWithLdap(
  username: string,
  password: string
): Promise<{ id: string; email: string; level: string } | null> {
  return new Promise((resolve) => {
    const client = ldap.createClient({ url: config.LDAP_URL })

    const userDn = `cn=${username},${config.LDAP_BASE_DN}`

    client.bind(userDn, password, (err) => {
      if (err) {
        client.destroy()
        resolve(null)
        return
      }

      // Rechercher les attributs de l'utilisateur
      const opts: ldap.SearchOptions = {
        filter: `(sAMAccountName=${username})`,
        scope: 'sub',
        attributes: ['objectGUID', 'mail', 'displayName', 'department', 'title'],
      }

      client.search(config.LDAP_BASE_DN, opts, (searchErr, res) => {
        if (searchErr) {
          client.destroy()
          resolve(null)
          return
        }

        const entries: ldap.SearchEntry[] = []
        res.on('searchEntry', (entry) => entries.push(entry))
        res.on('end', () => {
          client.destroy()
          if (entries.length === 0) { resolve(null); return }

          const entry = entries[0]!
          const email = String(entry.pojo.attributes.find((a) => a.type === 'mail')?.values[0] ?? '')
          // Mapper AD department/title vers AuthLevel (à adapter selon structure AD banque)
          const level = mapAdGroupToLevel(entry) ?? 'N6_CHARGE'

          resolve({ id: String(entry.pojo.objectName), email, level })
        })
      })
    })
  })
}

function mapAdGroupToLevel(entry: ldap.SearchEntry): string | null {
  // À adapter selon la structure Active Directory de la banque
  // Exemple: basé sur l'OU (Organizational Unit)
  const dn = entry.pojo.objectName
  if (dn.includes('OU=DG')) return 'N1_DG'
  if (dn.includes('OU=DGA')) return 'N2_DGA'
  if (dn.includes('OU=DirectionCentrale')) return 'N3_DIR_CENTRALE'
  if (dn.includes('OU=DirectionDept')) return 'N4_DIR_DEPT'
  if (dn.includes('OU=Agences,OU=Resp')) return 'N5_RESP_AGENCE'
  if (dn.includes('OU=Auditeurs')) return 'AUDITEUR'
  if (dn.includes('OU=RSSI')) return 'RSSI'
  return 'N6_CHARGE'
}
```

---

## 6. MICROSERVICES BACKEND (MS-1 à MS-6)

### 6.1 MS-1 : Service Dossier (25 JH)

**Objectif :** CRUD des dossiers, gestion des documents, recherche full-text.

**Démarrer avec Claude — Prompt recommandé :**
```
Contexte: Je développe MS-1 Dossier pour un parapheur digital de banque marocaine.
Stack: Node.js 20 + Fastify 4 + TypeScript + Prisma 5 + PostgreSQL 16 + Zod.
Spec: [coller le contenu de PHASE1-V2 section MS-1 et les US B.05 à B.12]

Génère:
1. La structure complète du dossier apps/ms-dossier/src/
2. Le service DossierService avec méthodes: create, findById, findAll (paginated), update, submit, cancel, duplicate
3. L'endpoint POST /dossiers avec validation Zod complète (montant, devise MAD/EUR/USD, type, contrepartie)
4. Les tests Vitest pour DossierService (100% coverage des cas nominaux + erreurs)

Contraintes:
- Pas d'any TypeScript
- Validation Zod sur toutes les entrées
- Chaque action inscrit dans l'audit log (via event BullMQ)
- Référence auto-générée: DOS-{YYYY}-{NNNN:04d}
- Conversion devise automatique (taux BAM cache Redis 1h)
```

**Structure MS-1 :**
```
apps/ms-dossier/src/
├── index.ts
├── app.ts
├── config.ts
├── routes/
│   ├── dossiers.create.ts    ← POST /dossiers
│   ├── dossiers.get.ts       ← GET /dossiers/:id
│   ├── dossiers.list.ts      ← GET /dossiers (paginated + search)
│   ├── dossiers.update.ts    ← PATCH /dossiers/:id
│   ├── dossiers.submit.ts    ← POST /dossiers/:id/submit
│   ├── dossiers.cancel.ts    ← POST /dossiers/:id/cancel
│   ├── documents.upload.ts   ← POST /dossiers/:id/documents
│   └── documents.download.ts ← GET /dossiers/:id/documents/:docId
├── services/
│   ├── DossierService.ts
│   ├── DocumentService.ts
│   ├── ReferenceGenerator.ts
│   └── DeviseConversionService.ts  ← appel API BAM + cache Redis
├── repositories/
│   ├── DossierRepository.ts
│   └── DocumentRepository.ts
└── __tests__/
    ├── DossierService.test.ts
    └── DeviseConversionService.test.ts
```

**Points clés d'implémentation MS-1 :**

```typescript
// services/ReferenceGenerator.ts
export async function generateReference(prisma: PrismaClient): Promise<string> {
  const year = new Date().getFullYear()
  const count = await prisma.dossier.count({
    where: { reference: { startsWith: `DOS-${year}-` } }
  })
  return `DOS-${year}-${String(count + 1).padStart(4, '0')}`
}

// services/DeviseConversionService.ts
export async function convertToMad(
  amount: number,
  devise: string,
  redis: Redis
): Promise<{ montantMad: number; tauxChange: number }> {
  if (devise === 'MAD') return { montantMad: amount, tauxChange: 1 }

  const cacheKey = `bam:rate:${devise}:MAD`
  const cached = await redis.get(cacheKey)

  if (cached) {
    const rate = parseFloat(cached)
    return { montantMad: amount * rate, tauxChange: rate }
  }

  // Appel API BAM
  const rate = await fetchBamRate(devise)
  await redis.setex(cacheKey, 3600, String(rate)) // cache 1h
  return { montantMad: amount * rate, tauxChange: rate }
}
```

### 6.2 MS-2 : Service Workflow — XState (40 JH)

**C'est le microservice le plus complexe. Planifier 2 semaines entières.**

**Prompt Claude pour XState machine :**
```
Contexte: Service Workflow pour parapheur digital banque marocaine.
Stack: Node.js 20 + TypeScript + XState 5 + Prisma 5.
Spec: voir PHASE2-V2 section workflows + 87 règles DMN + hiérarchie N1-N6.

Génère la machine XState pour le workflow de validation:
- États: brouillon → soumis → en_validation_N5 → en_validation_N4 → ... → approuvé / rejeté
- Events: SUBMIT, APPROVE, REJECT, REQUEST_INFO, RESPOND_INFO, ESCALATE, CANCEL
- Guards: vérification du niveau requis selon montant (87 règles DMN)
- Actions: créer ValidationTask, envoyer notification, calculer SLA deadline
- Services (invocations): appel API e-signature pour SEA/SEQ

La machine doit:
1. Être sérialisable en JSON (pour persistence PostgreSQL)
2. Reconstruire son état à partir du snapshot JSON
3. Calculer automatiquement les niveaux de validation requis via les règles DMN
4. Respecter les SLA (N5: 4h, N4: 8h, N3: 1j, N2: 2j, N1: 3j)
```

**Structure workflow XState :**

```typescript
// machines/dossierWorkflow.machine.ts
import { createMachine, assign } from 'xstate'

export const dossierWorkflowMachine = createMachine({
  id: 'dossierWorkflow',
  initial: 'brouillon',
  types: {} as {
    context: WorkflowContext
    events: WorkflowEvent
    guards: WorkflowGuards
  },
  context: ({ input }: { input: WorkflowInput }) => ({
    dossierId: input.dossierId,
    montantMad: input.montantMad,
    type: input.type,
    validationChain: computeValidationChain(input.montantMad, input.type),
    currentStep: 0,
    decisions: [],
    signatureRequests: [],
  }),

  states: {
    brouillon: {
      on: { SUBMIT: 'en_validation' },
    },

    en_validation: {
      entry: assign({ currentStep: 0 }),
      always: [
        { guard: 'validationComplete', target: 'approuve' },
        { target: 'attente_valideur' }
      ],
    },

    attente_valideur: {
      entry: 'assignToCurrentValideur',
      on: {
        APPROVE: { actions: 'recordApproval', target: 'en_validation' },
        REJECT: { target: 'rejete' },
        REQUEST_INFO: { target: 'en_attente_info' },
        ESCALATE: { actions: 'escalate', target: 'attente_valideur' },
      },
      after: {
        SLA_DELAY: { actions: 'triggerEscalation', target: 'attente_valideur' }
      }
    },

    en_attente_info: {
      on: {
        RESPOND_INFO: { target: 'attente_valideur' },
      }
    },

    approuve: {
      type: 'final',
      entry: 'notifyApproval',
    },

    rejete: {
      type: 'final',
      entry: 'notifyRejection',
    },
  },
})

// Règles DMN : calcul de la chaîne de validation
function computeValidationChain(
  montantMad: number,
  type: DossierType
): ValidationStep[] {
  const steps: ValidationStep[] = []

  // Règle R-SIGN-01: SES si montant < 100K et non-engagement
  if (montantMad < 100_000 && type !== 'ENGAGEMENT_CREDIT') {
    steps.push({ level: 'N5_RESP_AGENCE', signatureType: 'SES' })
    return steps
  }

  // Règle R-SIGN-02: SEA si montant 100K–5M et engagement
  if (montantMad >= 100_000 && montantMad < 5_000_000) {
    if (montantMad < 550_000) steps.push({ level: 'N5_RESP_AGENCE', signatureType: 'SEA' })
    if (montantMad >= 550_000) steps.push({ level: 'N4_DIR_DEPT', signatureType: 'SEA' })
    if (montantMad >= 2_200_000) steps.push({ level: 'N3_DIR_CENTRALE', signatureType: 'SEA' })
    return steps
  }

  // Règle R-SIGN-03: SEA+double si montant > 5M
  if (montantMad >= 5_000_000) {
    steps.push({ level: 'N4_DIR_DEPT', signatureType: 'SEA' })
    steps.push({ level: 'N3_DIR_CENTRALE', signatureType: 'SEA' })
    if (montantMad >= 11_000_000) steps.push({ level: 'N2_DGA', signatureType: 'SEA' })
    if (montantMad >= 50_000_000) steps.push({ level: 'N1_DG', signatureType: 'SEA' })
    return steps
  }

  return steps
}
```

### 6.3 MS-3 : Service Notifications (12 JH)

**Fonctions :** Email (SMTP banque), SMS (urgences), Push PWA, centre de notifications.

```typescript
// services/NotificationService.ts
export class NotificationService {
  constructor(
    private mailer: Transporter,
    private smsClient: SmsClient,
    private queue: Queue,
    private prisma: PrismaClient
  ) {}

  async sendValidationRequest(params: {
    taskId: string
    assigneeEmail: string
    dossierRef: string
    montant: string
    slaDeadline: Date
    langue: 'fr' | 'ar'
  }): Promise<void> {
    // Charger le template (bilingue)
    const template = await loadTemplate('validation-request', params.langue)

    // Email
    await this.queue.add('send-email', {
      to: params.assigneeEmail,
      subject: template.subject({ ref: params.dossierRef }),
      html: template.body(params),
    })

    // Log dans la base (centre notifications)
    await this.prisma.notification.create({
      data: {
        type: 'VALIDATION_REQUEST',
        taskId: params.taskId,
        recipientEmail: params.assigneeEmail,
        sentAt: new Date(),
        channel: 'email',
      },
    })
  }
}
```

**Templates email — structure bilingue :**
```
apps/ms-notification/src/templates/
├── fr/
│   ├── validation-request.html   ← nouvelle tâche de validation
│   ├── validation-approved.html  ← dossier approuvé
│   ├── validation-rejected.html  ← dossier rejeté
│   ├── sla-warning.html          ← avertissement délai 50%
│   ├── sla-breach.html           ← délai dépassé (escalade)
│   ├── delegation-created.html
│   └── info-requested.html
└── ar/
    ├── validation-request.html   ← même structure, RTL
    └── ...
```

### 6.4 MS-4 : Service Audit (15 JH)

**Règle absolue : append-only. Jamais de UPDATE ou DELETE sur audit_logs.**

```typescript
// services/AuditService.ts
import { createHash } from 'crypto'

export class AuditService {
  constructor(private prisma: PrismaClient) {}

  async log(entry: {
    entityType: string
    entityId: string
    action: string
    actorId?: string
    actorEmail?: string
    before?: unknown
    after?: unknown
    dossierId?: string
    ipAddress?: string
  }): Promise<void> {
    // Récupérer le dernier hash pour former la chaîne
    const lastLog = await this.prisma.auditLog.findFirst({
      orderBy: { id: 'desc' },
      select: { hash: true },
    })

    const payload = JSON.stringify({
      ...entry,
      timestamp: new Date().toISOString(),
    })

    // SHA-256(hash_précédent + payload) — intégrité de la chaîne
    const prevHash = lastLog?.hash ?? 'GENESIS'
    const hash = createHash('sha256')
      .update(prevHash + payload)
      .digest('hex')

    await this.prisma.auditLog.create({
      data: {
        entityType: entry.entityType,
        entityId: entry.entityId,
        action: entry.action,
        actorId: entry.actorId,
        actorEmail: entry.actorEmail,
        before: entry.before ? JSON.parse(JSON.stringify(entry.before)) : undefined,
        after: entry.after ? JSON.parse(JSON.stringify(entry.after)) : undefined,
        dossierId: entry.dossierId,
        ipAddress: entry.ipAddress,
        hash,
        prevHash: lastLog?.hash,
      },
    })
  }

  // Vérification d'intégrité de la chaîne (pour audit BAM)
  async verifyChainIntegrity(fromId: bigint, toId: bigint): Promise<{
    valid: boolean
    brokenAt?: bigint
  }> {
    const logs = await this.prisma.auditLog.findMany({
      where: { id: { gte: fromId, lte: toId } },
      orderBy: { id: 'asc' },
    })

    for (let i = 1; i < logs.length; i++) {
      const prev = logs[i - 1]!
      const curr = logs[i]!
      const expectedPrevHash = curr.prevHash

      if (expectedPrevHash !== prev.hash) {
        return { valid: false, brokenAt: curr.id }
      }
    }

    return { valid: true }
  }
}
```

### 6.5 MS-5/6 : Adapters e-signature et GED (30 JH)

**Pattern Anti-Corruption Layer : isoler l'application du format de l'API interne.**

```typescript
// apps/ms-esig-adapter/src/ports/ESignaturePort.ts
// Port: interface que l'application connaît
export interface ESignaturePort {
  createSignRequest(params: {
    dossierId: string
    documentId: string
    signerId: string
    signatureType: 'SES' | 'SEA' | 'SEQ'
    deadline: Date
  }): Promise<{ requestId: string; signUrl: string }>

  getSignRequestStatus(requestId: string): Promise<{
    status: 'pending' | 'signed' | 'rejected' | 'expired'
    signedAt?: Date
    certificate?: string
  }>

  handleCallback(payload: unknown): Promise<CallbackResult>
}

// apps/ms-esig-adapter/src/adapters/InternalESignAdapter.ts
// Adapter: traduit vers l'API réelle de la plateforme interne
export class InternalESignAdapter implements ESignaturePort {
  private client: AxiosInstance
  private circuitBreaker: CircuitBreaker

  constructor(private config: ESignConfig) {
    this.client = axios.create({
      baseURL: config.baseUrl,
      headers: { 'X-API-Key': config.apiKey },
      timeout: 30_000,
    })

    // Circuit breaker: 5 erreurs → open → mode dégradé (signature papier + scan)
    this.circuitBreaker = new CircuitBreaker(this.client, {
      threshold: 5,
      timeout: 30_000,
      resetTimeout: 120_000, // 2 min avant retry
      onOpen: () => this.notifyDegradedMode(),
    })
  }

  async createSignRequest(params: CreateSignRequestParams): Promise<SignRequestResult> {
    // Mapper vers le format de l'API interne (inconnu jusqu'à l'audit P0 Gate-1)
    const internalPayload = {
      // ← À adapter selon la doc de votre plateforme e-sig interne
      documentReference: params.documentId,
      signerEmployeeId: params.signerId,
      signatureClass: this.mapSignatureType(params.signatureType),
      expiresAt: params.deadline.toISOString(),
      callbackUrl: `${this.config.callbackUrl}?dossierId=${params.dossierId}`,
    }

    try {
      const response = await this.circuitBreaker.fire('post', '/sign-requests', internalPayload)
      return {
        requestId: String(response.data.id),
        signUrl: String(response.data.signingUrl),
      }
    } catch (err) {
      if (this.circuitBreaker.opened) {
        // Mode dégradé: enregistrer la nécessité de signature papier
        return this.createDegradedModeRequest(params)
      }
      throw err
    }
  }

  private mapSignatureType(type: 'SES' | 'SEA' | 'SEQ'): string {
    // Adapter selon le vocabulaire de la plateforme interne
    const map = { SES: 'simple', SEA: 'advanced', SEQ: 'qualified' }
    return map[type]
  }
}
```

---

## 7. FRONTEND REACT (INTERFACE UTILISATEUR)

### 7.1 Initialisation du projet React

```bash
cd apps/web
pnpm create vite . --template react-ts
pnpm add @tanstack/react-query @tanstack/react-query-devtools
pnpm add zustand
pnpm add react-hook-form @hookform/resolvers zod
pnpm add react-router-dom
pnpm add i18next react-i18next i18next-browser-languagedetector
pnpm add date-fns  # gestion dates FR/AR
pnpm add axios
pnpm add -D tailwindcss postcss autoprefixer
pnpm add -D @tailwindcss/forms @tailwindcss/typography
pnpm add -D vitest @testing-library/react @testing-library/user-event jsdom
pnpm add -D playwright @playwright/test

# shadcn/ui
npx shadcn-ui@latest init
# Sélectionner: TypeScript, Vite, Tailwind, default style, slate color
```

### 7.2 Configuration Tailwind RTL

**`tailwind.config.ts`** :
```typescript
import type { Config } from 'tailwindcss'

export default {
  content: ['./index.html', './src/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        // Palette Parapheur Digital
        primary: {
          50: '#EBF8FF',
          500: '#1A365D',  // bleu marine principal
          600: '#153050',
          700: '#0F2240',
        },
        accent: '#DD6B20',    // orange actions
        success: '#38A169',   // vert approbation
        danger: '#E53E3E',    // rouge risque/rejet
        warning: '#D69E2E',   // jaune alerte
      },
      fontFamily: {
        sans: ['Calibri', 'Segoe UI', 'sans-serif'],
        arabic: ['Cairo', 'Noto Sans Arabic', 'sans-serif'],
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
    require('tailwindcss-rtl'),  // support RTL
  ],
} satisfies Config
```

### 7.3 Configuration i18n FR/AR

**`src/i18n/index.ts`** :
```typescript
import i18n from 'i18next'
import { initReactI18next } from 'react-i18next'
import LanguageDetector from 'i18next-browser-languagedetector'
import fr from './locales/fr.json'
import ar from './locales/ar.json'

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    resources: { fr: { translation: fr }, ar: { translation: ar } },
    fallbackLng: 'fr',
    supportedLngs: ['fr', 'ar'],
    interpolation: { escapeValue: false },
  })

export default i18n
```

**`src/hooks/useDirection.ts`** :
```typescript
import { useTranslation } from 'react-i18next'
import { useEffect } from 'react'

export function useDirection() {
  const { i18n } = useTranslation()
  const dir = i18n.language === 'ar' ? 'rtl' : 'ltr'

  useEffect(() => {
    document.documentElement.dir = dir
    document.documentElement.lang = i18n.language
  }, [dir, i18n.language])

  return dir
}
```

**`src/i18n/locales/fr.json`** (extrait) :
```json
{
  "nav": {
    "dossiers": "Dossiers",
    "atValider": "À valider",
    "rapports": "Rapports"
  },
  "dossier": {
    "nouveau": "Nouveau dossier",
    "reference": "Référence",
    "montant": "Montant",
    "devise": "Devise",
    "contrepartie": "Contrepartie",
    "objet": "Objet",
    "type": {
      "ENGAGEMENT_CREDIT": "Engagement crédit",
      "CONTRAT_COMMERCIAL": "Contrat commercial",
      "DECISION_RH": "Décision RH",
      "ACHAT_INVESTISSEMENT": "Achat / Investissement"
    },
    "status": {
      "BROUILLON": "Brouillon",
      "SOUMIS": "Soumis",
      "EN_COURS": "En cours",
      "APPROUVE": "Approuvé",
      "REJETE": "Rejeté"
    }
  },
  "format": {
    "montant": "{{amount, number}} {{devise}}",
    "date": "dd/MM/yyyy",
    "dateHeure": "dd/MM/yyyy à HH:mm"
  },
  "actions": {
    "approuver": "Approuver",
    "rejeter": "Rejeter",
    "demanderInfo": "Demander info",
    "signer": "Signer",
    "soumettre": "Soumettre"
  }
}
```

### 7.4 Structure des pages

```
apps/web/src/
├── main.tsx                    ← point d'entrée
├── App.tsx                     ← router + providers
├── i18n/                       ← config i18n
├── pages/
│   ├── Login/
│   │   ├── LoginPage.tsx       ← formulaire LDAP + OTP
│   │   └── TotpPage.tsx        ← saisie code TOTP
│   ├── Dashboard/
│   │   ├── DashboardPage.tsx   ← vue d'ensemble (4 cards)
│   │   ├── components/
│   │   │   ├── StatCard.tsx
│   │   │   ├── TasksWidget.tsx  ← dossiers à valider
│   │   │   └── SlaAlerts.tsx   ← alertes SLA
│   ├── Dossiers/
│   │   ├── DossierListPage.tsx ← liste + filtres + recherche
│   │   ├── DossierCreatePage.tsx ← wizard création (4 étapes)
│   │   ├── DossierDetailPage.tsx ← détail + historique + actions
│   │   └── components/
│   │       ├── DossierCard.tsx
│   │       ├── DossierForm.tsx  ← React Hook Form + Zod
│   │       ├── DocumentUpload.tsx
│   │       ├── WorkflowTimeline.tsx ← visualisation étapes
│   │       └── SignatureModal.tsx   ← modal OTP signature
│   ├── Validation/
│   │   ├── ValidationQueuePage.tsx ← file d'attente valideur
│   │   └── components/
│   │       ├── ValidationCard.tsx
│   │       └── DecisionForm.tsx
│   ├── Reports/
│   │   ├── ReportsDashboardPage.tsx
│   │   └── BamReportPage.tsx   ← rapport BAM/AMMC auditeur
│   └── Admin/
│       ├── UsersPage.tsx
│       ├── DelegationsPage.tsx
│       └── AuditLogsPage.tsx
├── components/
│   ├── layout/
│   │   ├── AppLayout.tsx       ← sidebar + header
│   │   ├── Sidebar.tsx         ← navigation principale
│   │   └── Header.tsx          ← user menu + langue switcher
│   ├── ui/                     ← composants shadcn/ui
│   └── shared/
│       ├── MontantDisplay.tsx  ← format MAD/EUR/USD avec i18n
│       ├── DateDisplay.tsx     ← format dates FR/AR
│       ├── LevelBadge.tsx      ← badge N1-N6 coloré
│       ├── StatusBadge.tsx     ← badge statut dossier
│       └── SlaIndicator.tsx    ← barre progression SLA
├── hooks/
│   ├── useAuth.ts
│   ├── useDossiers.ts          ← TanStack Query hooks
│   ├── useValidationQueue.ts
│   └── useDirection.ts         ← RTL/LTR
├── stores/
│   ├── authStore.ts            ← Zustand auth state
│   └── notificationStore.ts
├── services/
│   ├── api.ts                  ← Axios client + interceptors
│   ├── dossierService.ts
│   └── workflowService.ts
└── types/                      ← types TS partagés (via @parapheur/shared-types)
```

### 7.5 Composant de création de dossier (wizard)

**Prompt Claude :**
```
Génère DossierCreatePage.tsx - wizard 4 étapes pour créer un dossier.

Étapes:
1. Type de dossier + montant + devise (MAD/EUR/USD) + contrepartie lookup
2. Informations détaillées (objet, date besoin, urgence flag)
3. Upload documents (drag & drop, validation MIME, max 50MB)
4. Récapitulatif + soumettre

Contraintes:
- React Hook Form + Zod validation step-by-step
- shadcn/ui pour les composants (Dialog, Form, Input, Select, Button, Progress)
- TanStack Query pour le lookup contrepartie (debounce 300ms)
- i18n FR/AR complet
- Feedback visuel: calcul montant MAD temps réel si devise EUR/USD
- Désactiver "Suivant" si étape invalide
- Loading state pendant la soumission
```

### 7.6 Modal de signature

```typescript
// components/shared/SignatureModal.tsx
import { useState, useEffect } from 'react'
import { Dialog, DialogContent, DialogHeader, DialogTitle } from '../ui/dialog'
import { InputOTP } from '../ui/input-otp'
import { useTranslation } from 'react-i18next'

interface SignatureModalProps {
  open: boolean
  onClose: () => void
  onSign: (otp: string) => Promise<void>
  signatureType: 'SES' | 'SEA' | 'SEQ'
  dossierRef: string
  montantMad: number
}

export function SignatureModal({
  open, onClose, onSign, signatureType, dossierRef, montantMad
}: SignatureModalProps) {
  const { t } = useTranslation()
  const [otp, setOtp] = useState('')
  const [countdown, setCountdown] = useState(300) // 5 minutes
  const [loading, setLoading] = useState(false)

  useEffect(() => {
    if (!open) { setCountdown(300); setOtp(''); return }
    const timer = setInterval(() => {
      setCountdown(c => { if (c <= 1) { clearInterval(timer); return 0 } return c - 1 })
    }, 1000)
    return () => clearInterval(timer)
  }, [open])

  const formatCountdown = (s: number) => `${String(Math.floor(s / 60)).padStart(2, '0')}:${String(s % 60).padStart(2, '0')}`

  return (
    <Dialog open={open} onOpenChange={onClose}>
      <DialogContent className="max-w-sm">
        <DialogHeader>
          <DialogTitle className="text-center">
            {t('signature.titre', { type: signatureType })}
          </DialogTitle>
        </DialogHeader>

        <div className="space-y-4">
          {/* Récapitulatif sécurisé */}
          <div className="rounded-lg bg-slate-50 p-4 text-sm space-y-1">
            <p><span className="text-slate-500">{t('dossier.reference')} :</span> <strong>{dossierRef}</strong></p>
            <p><span className="text-slate-500">{t('dossier.montant')} :</span> <strong className="text-accent">{montantMad.toLocaleString('fr-MA')} MAD</strong></p>
            <p><span className="text-slate-500">{t('signature.type')} :</span> <span className="font-medium text-primary-500">{signatureType}</span></p>
          </div>

          {/* Code OTP */}
          <div className="space-y-2">
            <p className="text-sm text-center text-slate-600">{t('signature.saisirCode')}</p>
            <div className="flex justify-center">
              <InputOTP value={otp} onChange={setOtp} maxLength={6} />
            </div>
            <p className="text-center text-xs text-slate-500">
              {t('signature.expirationCode')} : <span className={countdown < 60 ? 'text-danger font-medium' : ''}>{formatCountdown(countdown)}</span>
            </p>
          </div>

          {/* Actions */}
          <div className="flex gap-2">
            <button onClick={onClose} className="flex-1 btn-secondary">{t('actions.annuler')}</button>
            <button
              disabled={otp.length !== 6 || loading || countdown === 0}
              onClick={async () => { setLoading(true); try { await onSign(otp) } finally { setLoading(false) } }}
              className="flex-1 btn-primary"
            >
              {loading ? t('actions.traitement') : t('actions.signer')}
            </button>
          </div>
        </div>
      </DialogContent>
    </Dialog>
  )
}
```

---

## 8. INTÉGRATIONS EXTERNES (E-SIGNATURE, GED, AD)

### 8.1 Checklist d'audit préalable API e-signature

**À faire en Phase P0 (Gate-1) — avant tout développement MS-5 :**

```
Audit API plateforme e-signature interne:
[ ] Format d'authentification: API Key? OAuth2? Certificat client?
[ ] Endpoint création demande: POST /? Payload format?
[ ] Endpoint statut: GET /? Polling ou callback?
[ ] Format callback: webhook? Quels événements?
[ ] Type de signature supportée: SES seul? SEA? SEQ avec DGSSI?
[ ] Agrément DGSSI (Loi 43-20 art. 17): oui / non / en cours?
[ ] Environnement de test disponible?
[ ] Rate limits?
[ ] SLA de disponibilité?
[ ] Contact technique DSI responsable?
```

**Rédiger un document `docs/api/esig-audit.md`** pour formaliser les réponses.

### 8.2 Variables d'intégration à configurer

```typescript
// packages/shared-config/integrations.ts
export const INTEGRATION_POINTS = {
  ESIG: {
    endpoints: {
      // À remplir après audit
      createRequest: `${process.env.ESIG_BASE_URL}/ENDPOINT_A_DEFINIR`,
      getStatus: `${process.env.ESIG_BASE_URL}/ENDPOINT_A_DEFINIR/:id`,
      callback: process.env.ESIG_CALLBACK_URL,
    },
    signatureTypes: {
      SES: 'VALEUR_A_DEFINIR',  // selon nomenclature interne
      SEA: 'VALEUR_A_DEFINIR',
      SEQ: 'VALEUR_A_DEFINIR',
    },
  },
  GED: {
    endpoints: {
      uploadDocument: `${process.env.GED_BASE_URL}/ENDPOINT_A_DEFINIR`,
      downloadDocument: `${process.env.GED_BASE_URL}/ENDPOINT_A_DEFINIR/:id`,
      setRetention: `${process.env.GED_BASE_URL}/ENDPOINT_A_DEFINIR/:id/retention`,
    },
    retentionPolicy: {
      default: 10,   // 10 ans (BAM 5/W/2017)
      rh: 5,
      fiscal: 7,     // Code général des impôts Maroc
    },
  },
} as const
```

### 8.3 Tests d'intégration avec mocks

**Ne jamais intégrer en direct sans tests. Utiliser des mocks pour les API externes :**

```typescript
// apps/ms-esig-adapter/src/__tests__/ESignAdapter.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { InternalESignAdapter } from '../adapters/InternalESignAdapter.js'
import axios from 'axios'

vi.mock('axios')

describe('InternalESignAdapter', () => {
  let adapter: InternalESignAdapter

  beforeEach(() => {
    adapter = new InternalESignAdapter({
      baseUrl: 'https://esig-test.banque.local/api',
      apiKey: 'test-key',
      callbackUrl: 'http://localhost:3005/callbacks',
    })
  })

  it('crée une demande de signature SEA avec succès', async () => {
    vi.mocked(axios.post).mockResolvedValueOnce({
      data: { id: 'req-123', signingUrl: 'https://esig.banque.local/sign/req-123' },
    })

    const result = await adapter.createSignRequest({
      dossierId: 'dos-001',
      documentId: 'doc-001',
      signerId: 'emp-001',
      signatureType: 'SEA',
      deadline: new Date('2026-05-01'),
    })

    expect(result.requestId).toBe('req-123')
    expect(result.signUrl).toContain('req-123')
  })

  it('active le mode dégradé après 5 erreurs consécutives', async () => {
    vi.mocked(axios.post).mockRejectedValue(new Error('Connection refused'))

    // 5 tentatives → circuit breaker s'ouvre
    for (let i = 0; i < 5; i++) {
      await expect(adapter.createSignRequest({
        dossierId: `dos-00${i}`,
        documentId: 'doc-001',
        signerId: 'emp-001',
        signatureType: 'SEA',
        deadline: new Date(),
      })).rejects.toThrow()
    }

    expect(adapter.isInDegradedMode()).toBe(true)
  })
})
```

---

## 9. TESTS ET QUALITÉ (TDD/BDD)

### 9.1 Stratégie de tests

```
Tests unitaires (Vitest)          ← services, utils, règles DMN
Tests intégration (Vitest + Prisma) ← repositories, workflows
Tests e2e (Playwright)            ← parcours utilisateurs complets
Tests contrats (OpenAPI)          ← validation des contrats API
Tests sécurité (OWASP ZAP)        ← DNSSI compliance
```

**Objectif couverture :** 80% minimum (gate CI/CD) — 95% sur MS-2 Workflow.

### 9.2 Configuration Vitest

**`vitest.config.ts`** (dans chaque app) :
```typescript
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 75,
        statements: 80,
      },
      exclude: [
        'src/index.ts',
        'src/**/*.d.ts',
        'prisma/**',
      ],
    },
    setupFiles: ['./src/__tests__/setup.ts'],
  },
})
```

### 9.3 Tests des règles DMN (87 règles)

```typescript
// packages/shared-schemas/src/__tests__/dmnRules.test.ts
import { describe, it, expect } from 'vitest'
import { computeValidationChain } from '../dmnRules.js'

describe('Règles DMN — Niveaux de signature (Loi 43-20)', () => {
  describe('R-SIGN-01: SES pour montants < 100K non-engagement', () => {
    it('dossier RH 50K MAD → SES N5 uniquement', () => {
      const chain = computeValidationChain(50_000, 'DECISION_RH')
      expect(chain).toHaveLength(1)
      expect(chain[0]!.signatureType).toBe('SES')
      expect(chain[0]!.level).toBe('N5_RESP_AGENCE')
    })

    it('dossier achat 80K MAD → SES N5 uniquement', () => {
      const chain = computeValidationChain(80_000, 'ACHAT_INVESTISSEMENT')
      expect(chain).toHaveLength(1)
      expect(chain[0]!.signatureType).toBe('SES')
    })
  })

  describe('R-SIGN-02: SEA pour engagements crédit 100K–5M', () => {
    it('crédit 300K MAD → SEA N5', () => {
      const chain = computeValidationChain(300_000, 'ENGAGEMENT_CREDIT')
      expect(chain).toHaveLength(1)
      expect(chain[0]!.signatureType).toBe('SEA')
      expect(chain[0]!.level).toBe('N5_RESP_AGENCE')
    })

    it('crédit 800K MAD → SEA N4', () => {
      const chain = computeValidationChain(800_000, 'ENGAGEMENT_CREDIT')
      expect(chain).toHaveLength(1)
      expect(chain[0]!.level).toBe('N4_DIR_DEPT')
    })

    it('crédit 3M MAD → SEA N4 + N3', () => {
      const chain = computeValidationChain(3_000_000, 'ENGAGEMENT_CREDIT')
      expect(chain).toHaveLength(2)
      expect(chain[0]!.level).toBe('N4_DIR_DEPT')
      expect(chain[1]!.level).toBe('N3_DIR_CENTRALE')
    })
  })

  describe('R-SIGN-03: Double signature > 5M MAD', () => {
    it('crédit 10M MAD → SEA N4 + N3 + N2', () => {
      const chain = computeValidationChain(10_000_000, 'ENGAGEMENT_CREDIT')
      expect(chain).toHaveLength(3)
      expect(chain[2]!.level).toBe('N2_DGA')
    })

    it('crédit 60M MAD → chaîne complète jusqu\'au DG', () => {
      const chain = computeValidationChain(60_000_000, 'ENGAGEMENT_CREDIT')
      expect(chain[chain.length - 1]!.level).toBe('N1_DG')
    })
  })
})
```

### 9.4 Tests e2e Playwright

```typescript
// apps/web/e2e/dossier-creation.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Création de dossier — Parcours Khalid (N6)', () => {
  test.beforeEach(async ({ page }) => {
    // Connexion test
    await page.goto('/login')
    await page.fill('[name=username]', 'khalid.test')
    await page.fill('[name=password]', 'Test@1234')
    await page.click('button[type=submit]')
    // Passer TOTP (désactivé en env test)
    await expect(page).toHaveURL('/dashboard')
  })

  test('créer un dossier crédit 800K MAD et le soumettre', async ({ page }) => {
    await page.click('[data-testid=btn-nouveau-dossier]')

    // Étape 1: Type et montant
    await page.selectOption('[name=type]', 'ENGAGEMENT_CREDIT')
    await page.fill('[name=montant]', '800000')
    await page.selectOption('[name=devise]', 'MAD')
    await page.fill('[name=contrepartie]', 'Société TEST')
    // Vérifier que le montant MAD s'affiche correctement
    await expect(page.locator('[data-testid=montant-mad]')).toContainText('800 000,00 MAD')
    await page.click('[data-testid=btn-suivant]')

    // Étape 2: Détails
    await page.fill('[name=objet]', 'Ligne de crédit exploitation - renouvellement')
    await page.click('[data-testid=btn-suivant]')

    // Étape 3: Documents (upload test)
    await page.setInputFiles('[name=documents]', 'e2e/fixtures/contrat-test.pdf')
    await expect(page.locator('[data-testid=document-uploaded]')).toBeVisible()
    await page.click('[data-testid=btn-suivant]')

    // Étape 4: Récapitulatif
    await expect(page.locator('[data-testid=recap-montant]')).toContainText('800 000')
    await expect(page.locator('[data-testid=recap-signature]')).toContainText('SEA')

    // Soumettre
    await page.click('[data-testid=btn-soumettre]')
    await expect(page.locator('[data-testid=toast-success]')).toContainText('Dossier soumis')

    // Vérifier la référence générée
    await expect(page.locator('[data-testid=dossier-reference]')).toMatch(/DOS-2026-\d{4}/)
  })

  test('affichage RTL correct en langue arabe', async ({ page }) => {
    // Changer la langue
    await page.click('[data-testid=lang-switcher]')
    await page.click('[data-testid=lang-ar]')

    // Vérifier l'attribut dir
    await expect(page.locator('html')).toHaveAttribute('dir', 'rtl')

    // Vérifier que le formulaire est bien en RTL
    const form = page.locator('[data-testid=dossier-form]')
    await expect(form).toHaveCSS('direction', 'rtl')
  })
})
```

---

## 10. SÉCURITÉ ET CONFORMITÉ (DNSSI/ISO 27001/LOI 43-20)

### 10.1 Checklist sécurité obligatoire (avant mise en production)

```
AUTHENTIFICATION & AUTORISATION
[ ] LDAP/AD : bind service account avec droits read-only uniquement
[ ] JWT : durée 8h, refresh token rotation, révocation stockée Redis
[ ] TOTP : issuer banque + QR code enrollment + codes de secours chiffrés
[ ] RBAC : chaque route vérifie le niveau AuthLevel requis
[ ] Sessions : HttpOnly cookies, SameSite=Strict, Secure (HTTPS seulement)

PROTECTION DES DONNÉES (Loi 09-08)
[ ] Chiffrement en transit : TLS 1.3 uniquement (TLS 1.0/1.1 désactivé)
[ ] Chiffrement au repos : PostgreSQL tablespace chiffré (AES-256)
[ ] PII inventory : 25 champs identifiés, minimisation des données
[ ] Logs : pas de données personnelles dans les logs (masquage)
[ ] Droit d'accès : endpoint GET /api/v1/me/data (Loi 09-08 art. 7)
[ ] Suppression : endpoint DELETE /api/v1/me/data (art. 7)

SÉCURITÉ APPLICATIVE (OWASP Top 10 + STRIDE)
[ ] SQL Injection : Prisma paramétré (pas de concaténation SQL)
[ ] XSS : Content-Security-Policy strict + DOMPurify pour contenu user
[ ] CSRF : tokens CSRF sur toutes les mutations
[ ] Rate limiting : /auth max 5 req/5min, /api max 100/min
[ ] Upload : validation MIME type + taille (max 50MB) + virus scan
[ ] Headers : HSTS + X-Frame-Options DENY + X-Content-Type-Options
[ ] Secrets : jamais dans le code, variables d'env + Vault

AUDIT ET TRAÇABILITÉ (BAM + AMMC)
[ ] Audit log : chaque action enregistrée (qui, quoi, quand, depuis où)
[ ] Hash chain : intégrité SHA-256 vérifiable
[ ] Rétention : 10 ans minimum (BAM 5/W/2017)
[ ] Export : rapport exportable pour BAM/AMMC sur demande

DISPONIBILITÉ (BAM 5/W/2017: RTO≤4h, RPO≤1h)
[ ] Sauvegarde PostgreSQL : pg_dump quotidien + WAL archiving (RPO 1h)
[ ] Test de restore : test mensuel documenté
[ ] Monitoring : alertes sur CPU >80%, DB connections >80%, erreurs >1%
[ ] PRA documenté : procédure de recovery testée
```

### 10.2 Middleware d'autorisation

```typescript
// apps/api-gateway/src/middleware/authorize.ts
import { FastifyRequest, FastifyReply } from 'fastify'

type AuthLevel = 'N1_DG' | 'N2_DGA' | 'N3_DIR_CENTRALE' | 'N4_DIR_DEPT' | 'N5_RESP_AGENCE' | 'N6_CHARGE' | 'AUDITEUR' | 'RSSI' | 'ADMIN'

const LEVEL_ORDER: Record<AuthLevel, number> = {
  N1_DG: 6, N2_DGA: 5, N3_DIR_CENTRALE: 4,
  N4_DIR_DEPT: 3, N5_RESP_AGENCE: 2, N6_CHARGE: 1,
  AUDITEUR: 0, RSSI: 0, ADMIN: 10,
}

export function requireLevel(minLevel: AuthLevel) {
  return async (request: FastifyRequest, reply: FastifyReply) => {
    const user = request.user as { level: AuthLevel; sub: string }

    if (!user || LEVEL_ORDER[user.level] < LEVEL_ORDER[minLevel]) {
      return reply.code(403).send({
        statusCode: 403,
        error: 'Forbidden',
        message: `Niveau requis : ${minLevel}. Votre niveau : ${user?.level ?? 'non authentifié'}`,
      })
    }
  }
}

export function requireAuditeur() {
  return async (request: FastifyRequest, reply: FastifyReply) => {
    const user = request.user as { level: AuthLevel }
    if (user.level !== 'AUDITEUR' && user.level !== 'ADMIN') {
      return reply.code(403).send({ statusCode: 403, error: 'Reserved for auditors' })
    }
  }
}
```

### 10.3 Déclaration CNDP (Loi 09-08)

**`docs/cndp/procedure-declaration.md`** — Actions à réaliser :
```
1. NOMMER un correspondant données personnelles interne
   → Formulaire CNDP : https://www.cndp.ma (section Déclarations)

2. CONSTITUER le dossier de déclaration:
   → Description du traitement (parapheur, gestion dossiers)
   → Finalité précise (validation engagement bancaire)
   → Données traitées (nom, matricule, email, IP, actions)
   → Durée de conservation (10 ans BAM)
   → Destinataires (auditeurs internes, BAM sur demande)
   → Mesures de sécurité (TLS, chiffrement, contrôle d'accès)

3. DÉPOSER la déclaration
   → Délai d'instruction: 2 mois
   → Frais: 5 000 MAD (prévoir dans budget CAPEX)

4. OBTENIR le récépissé avant mise en production

5. AFFICHER la politique de confidentialité (Loi 09-08 art. 5)
   → Route publique: GET /privacy-policy
   → Bilingue FR/AR
```

---

## 11. CI/CD ET DÉPLOIEMENT

### 11.1 Pipeline GitHub Actions

**`.github/workflows/ci.yml`** :
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  DATABASE_URL: postgresql://parapheur:password@localhost:5432/parapheur_test
  REDIS_URL: redis://localhost:6379

jobs:
  quality:
    name: Code Quality
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: 9 }
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'pnpm' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm run format --check

  test:
    name: Tests
    runs-on: ubuntu-latest
    needs: quality

    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: parapheur
          POSTGRES_PASSWORD: password
          POSTGRES_DB: parapheur_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      redis:
        image: redis:7-alpine
        options: --health-cmd "redis-cli ping" --health-interval 10s

    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: 9 }
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'pnpm' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm --filter @parapheur/db prisma migrate deploy
      - run: pnpm test -- --coverage

      # Gate: couverture minimum 80%
      - name: Coverage gate
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% < 80% - Build failed"
            exit 1
          fi

  security:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: quality
    steps:
      - uses: actions/checkout@v4
      - name: SAST (Semgrep)
        uses: semgrep/semgrep-action@v1
        with:
          config: >-
            p/typescript
            p/nodejs
            p/owasp-top-ten

      - name: Dependency audit
        run: pnpm audit --audit-level high

      - name: Container scan (Trivy)
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          severity: 'CRITICAL,HIGH'

  build:
    name: Build Docker Images
    runs-on: ubuntu-latest
    needs: [test, security]
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - name: Build images
        run: docker-compose -f docker-compose.prod.yml build
      - name: Tag and push
        run: |
          docker tag parapheur-api-gateway:latest registry.banque.local/parapheur/api-gateway:${{ github.sha }}
          docker push registry.banque.local/parapheur/api-gateway:${{ github.sha }}

  deploy-staging:
    name: Deploy Staging
    runs-on: ubuntu-latest
    needs: build
    environment: staging
    steps:
      - name: Deploy to staging
        run: |
          ssh deploy@staging.banque.local "
            cd /opt/parapheur &&
            docker-compose -f docker-compose.prod.yml pull &&
            docker-compose -f docker-compose.prod.yml up -d --no-build
          "
```

### 11.2 Dockerfiles de production

**`infra/docker/Dockerfile.node`** (base pour tous les microservices Node) :
```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install --frozen-lockfile
COPY . .
RUN pnpm build

# Production stage
FROM node:20-alpine AS runner
WORKDIR /app
RUN addgroup -S parapheur && adduser -S parapheur -G parapheur

COPY --from=builder --chown=parapheur:parapheur /app/dist ./dist
COPY --from=builder --chown=parapheur:parapheur /app/node_modules ./node_modules
COPY --from=builder --chown=parapheur:parapheur /app/package.json .

USER parapheur
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1

CMD ["node", "dist/index.js"]
```

### 11.3 Stratégie de déploiement — Rolling Update

```yaml
# deploy/k8s/api-gateway.yaml (si Kubernetes) ou docker-compose.prod.yml
services:
  api-gateway:
    image: registry.banque.local/parapheur/api-gateway:${VERSION}
    deploy:
      replicas: 2
      update_config:
        parallelism: 1          # déployer un à la fois
        delay: 30s              # attendre 30s entre chaque
        order: start-first      # démarrer le nouveau avant d'arrêter l'ancien
        failure_action: rollback
      rollback_config:
        parallelism: 1
        delay: 30s
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

---

## 12. OPÉRATIONS ET MONITORING

### 12.1 Métriques Prometheus

```typescript
// packages/shared-utils/src/metrics.ts
import { Registry, Counter, Histogram, Gauge } from 'prom-client'

export const metrics = {
  httpRequests: new Counter({
    name: 'parapheur_http_requests_total',
    help: 'Total HTTP requests',
    labelNames: ['method', 'route', 'status'],
  }),

  httpDuration: new Histogram({
    name: 'parapheur_http_duration_seconds',
    help: 'HTTP request duration',
    labelNames: ['method', 'route'],
    buckets: [0.1, 0.3, 0.5, 1, 2, 5],
  }),

  dossierCreated: new Counter({
    name: 'parapheur_dossiers_created_total',
    help: 'Total dossiers created',
    labelNames: ['type'],
  }),

  workflowStep: new Counter({
    name: 'parapheur_workflow_transitions_total',
    help: 'Workflow state transitions',
    labelNames: ['from_state', 'to_state', 'decision'],
  }),

  slaBreaches: new Counter({
    name: 'parapheur_sla_breaches_total',
    help: 'SLA deadline breaches',
    labelNames: ['level'],
  }),

  eSignAdapterStatus: new Gauge({
    name: 'parapheur_esig_circuit_breaker_open',
    help: 'E-signature circuit breaker state (1=open/degraded)',
  }),

  pendingTasks: new Gauge({
    name: 'parapheur_pending_validation_tasks',
    help: 'Currently pending validation tasks',
    labelNames: ['level'],
  }),
}
```

### 12.2 Alertes Grafana (Alertmanager)

```yaml
# infra/monitoring/alerts.yaml
groups:
  - name: parapheur-alerts
    rules:
      - alert: ESignAdapterDown
        expr: parapheur_esig_circuit_breaker_open == 1
        for: 5m
        labels: { severity: critical }
        annotations:
          summary: "Plateforme e-signature INACCESSIBLE"
          description: "Circuit breaker ouvert depuis 5 min. Mode dégradé (signature papier) activé."

      - alert: SlaBreachRate
        expr: rate(parapheur_sla_breaches_total[1h]) > 5
        for: 10m
        labels: { severity: warning }
        annotations:
          summary: "Taux de dépassement SLA élevé"
          description: "Plus de 5 dépassements SLA en 1 heure."

      - alert: DatabaseConnections
        expr: pg_stat_database_numbackends / pg_settings_max_connections > 0.8
        for: 5m
        labels: { severity: warning }
        annotations:
          summary: "PostgreSQL > 80% connexions utilisées"

      - alert: HighErrorRate
        expr: rate(parapheur_http_requests_total{status=~"5.."}[5m]) > 0.01
        for: 5m
        labels: { severity: critical }
        annotations:
          summary: "Taux d'erreur HTTP 5xx > 1%"
```

---

## 13. WORKFLOW CLAUDE — COMMENT TRAVAILLER AVEC L'IA

### 13.1 Principes fondamentaux

Claude est votre **pair programmer**. Pour obtenir un code professionnel, respecter ces règles :

| Situation | Approche Claude |
|-----------|-----------------|
| Nouveau microservice | Nouvelle conversation + coller la spec Phase correspondante |
| Débogage erreur | Coller l'erreur complète + le code concerné |
| Review de code | Coller le fichier + demander "code review professionnel" |
| Tests manquants | Coller le service + demander "génère les tests Vitest" |
| Question architecture | Coller le contexte projet + la question précise |

### 13.2 Prompts types à réutiliser

**Prompt: Générer un microservice**
```
Contexte technique:
- Stack: Node.js 20 + Fastify 4 + TypeScript 5 + Prisma 5 + Zod + Vitest
- Pattern: Repository + Service + Route (3 couches)
- Conventions: pas d'any, strict TypeScript, ESLint + Prettier configurés

Spec fonctionnelle: [COLLER LE CONTENU DE LA SPEC CORRESPONDANTE]

Génère le microservice [NOM] avec:
1. Structure de dossiers complète
2. Schema Prisma si nouveau modèle
3. Repository avec CRUD + pagination
4. Service avec logique métier
5. Routes Fastify avec schéma Zod
6. Tests Vitest (coverage 80% minimum)

Contraintes:
- Chaque mutation publie un event BullMQ pour l'audit
- Gestion d'erreurs avec codes HTTP appropriés (400, 401, 403, 404, 422, 500)
- Logging Pino structuré (JSON)
- Pas de console.log
```

**Prompt: Code review**
```
Fais une code review professionnelle de ce fichier TypeScript.
Critères: sécurité (OWASP), performance, maintenabilité, couverture d'erreurs, conventions TS.
Indique les problèmes par niveau: CRITIQUE / MAJEUR / MINEUR / SUGGESTION.

[COLLER LE CODE]
```

**Prompt: Débogage**
```
J'ai cette erreur: [COLLER L'ERREUR COMPLÈTE avec stacktrace]

Contexte:
- Fichier: [chemin/fichier.ts]
- Que j'essayais de faire: [description]
- Code concerné: [COLLER LE CODE]

Analyse la cause racine et propose la correction.
```

**Prompt: Tests manquants**
```
Voici le service [Nom] avec ses méthodes:
[COLLER LE CODE DU SERVICE]

Génère les tests Vitest complets:
- Tests nominaux (happy path)
- Tests cas d'erreur (invalid input, not found, unauthorized)
- Tests limites (montants à la frontière des règles DMN)
- Mocks pour Prisma + Redis + API externes
- Coverage cible: 95% pour ce service critique
```

### 13.3 Organisation des conversations Claude

```
CONVERSATION 1: "Init projet + DB"
→ monorepo setup, docker-compose, schema Prisma, migrations

CONVERSATION 2: "API Gateway + Auth LDAP"
→ Fastify app, plugins, routes auth, TOTP

CONVERSATION 3: "MS-1 Dossier"
→ CRUD dossiers, documents, référence, conversion devise

CONVERSATION 4: "MS-2 Workflow XState"
→ machine XState, règles DMN, SLA, escalade (session longue)

CONVERSATION 5: "MS-3/4 Notifications + Audit"
→ templates email/SMS, BullMQ jobs, audit chain

CONVERSATION 6: "MS-5/6 Adapters e-sig + GED"
→ Anti-corruption layer, circuit breaker, mocks API

CONVERSATION 7: "Frontend — Auth + Dashboard"
→ Login page, TOTP, dashboard, navigation

CONVERSATION 8: "Frontend — Dossiers CRUD"
→ Liste, wizard création, détail, upload documents

CONVERSATION 9: "Frontend — Workflow UI"
→ Queue validation, modal signature, timeline

CONVERSATION 10: "Frontend — i18n + RTL"
→ Traductions FR/AR, RTL layout, formats locaux

CONVERSATION 11: "Tests e2e Playwright"
→ Parcours complets Khalid, Aïcha, Mohammed, Yassir

CONVERSATION 12: "Sécurité + CI/CD"
→ Gates CI/CD, Dockerfiles, scanning, déploiement
```

### 13.4 Git workflow en solo

```bash
# Feature branch par US ou composant
git checkout -b feat/us-05-creation-dossier

# Commits atomiques avec convention
git commit -m "feat(ms-dossier): add POST /dossiers with Zod validation"
git commit -m "test(ms-dossier): add 95% coverage for DossierService"
git commit -m "fix(ms-dossier): handle devise conversion error gracefully"

# PR self-review avant merge
gh pr create --title "US-05: Création dossier" --body "..."
# Faire review avec Claude sur le PR diff
# Merger après validation

# Tags pour jalons
git tag -a v0.1.0 -m "MVP: MS-1 Dossier complet"
git tag -a v0.2.0 -m "MVP: MS-2 Workflow complet"
```

---

## 14. CHECKLIST DE LIVRAISON

### 14.1 Checklist avant mise en production (go-live)

```
FONCTIONNEL
[ ] Les 47 User Stories des 10 epics sont implementées
[ ] Tests e2e Playwright : 5 parcours utilisateurs validés
[ ] Tests unitaires : couverture ≥ 80% (tous microservices)
[ ] Tests DMN : 87 règles testées avec assertions
[ ] Performance : temps de réponse < 2s (P95) pour 50 utilisateurs simultanés
[ ] i18n : interface complète en FR et AR, RTL correct

SÉCURITÉ
[ ] Scan SAST : 0 vulnérabilité CRITIQUE ou HIGH
[ ] Dépendances : pnpm audit 0 HIGH+
[ ] Scan Docker : Trivy 0 CRITICAL
[ ] OWASP ZAP : rapport fourni au RSSI
[ ] Test de pénétration (optionnel mais recommandé) : rapport
[ ] Review code sécurité : RSSI a validé l'architecture

CONFORMITÉ RÉGLEMENTAIRE
[ ] CNDP : récépissé de déclaration obtenu
[ ] Politique de confidentialité : en ligne (/privacy-policy), bilingue
[ ] Loi 43-20 : type de signature validé par DSI + RSSI (agrément DGSSI?)
[ ] BAM 5/W/2017 : audit log 10 ans configuré, test de restore documenté
[ ] Loi 05-20 OIV : dossier sécurité soumis à DGSSI (si applicable)

OPÉRATIONS
[ ] Monitoring : Prometheus + Grafana opérationnel
[ ] Alertes : configurées et testées (e-sig down, SLA breach, DB)
[ ] Sauvegarde : pg_dump quotidien + WAL archiving + test de restore
[ ] PRA : procédure documentée + test de recovery
[ ] Runbook : guide opérateur de production rédigé
[ ] Rollback : procédure de rollback testée

FORMATION
[ ] Guide utilisateur : rédigé (FR + AR)
[ ] Formation valideurs : session réalisée (N3-N5)
[ ] Formation admins : session réalisée (RSSI + DSI)
[ ] Support : process de ticketing configuré

ACCEPTATION
[ ] Recette métier : UAT signé par représentants N6, N4, N2, Auditeur
[ ] Go-live : validé DG + DSI + RSSI + DPO
```

### 14.2 Indicateurs clés de succès

| KPI | Cible | Mesure |
|-----|-------|--------|
| Temps de traitement moyen | < 6h (vs 8 jours papier) | Dashboard direction |
| Taux de respect SLA | > 95% | Alerte Grafana |
| Disponibilité système | > 99.5% | Uptime monitoring |
| Couverture tests | > 80% | Rapport CI/CD |
| Dossiers traités J1 | ≥ 50 | Dashboard pilotage |
| Score satisfaction utilisateurs | > 7/10 | Enquête semaine 2 |

---

## ANNEXE A — Commandes quotidiennes utiles

```bash
# Démarrer l'environnement de développement
pnpm docker:up && pnpm dev

# Créer une nouvelle migration Prisma
pnpm db:migrate --name "add_notification_table"

# Générer les types Prisma après modification du schema
pnpm db:generate

# Lancer les tests en watch mode
pnpm --filter ms-workflow test --watch

# Voir les logs d'un microservice
docker-compose logs -f ms-workflow

# Accéder à Prisma Studio (interface visuelle PostgreSQL)
pnpm db:studio

# Vérifier les emails envoyés (Maildev)
open http://localhost:1080

# Lancer Playwright en mode debug (interface graphique)
pnpm --filter web playwright test --debug

# Générer le rapport de couverture
pnpm test -- --coverage && open coverage/index.html
```

## ANNEXE B — Ressources clés

| Ressource | URL |
|-----------|-----|
| Fastify docs | https://fastify.dev |
| Prisma docs | https://prisma.io/docs |
| XState 5 docs | https://stately.ai/docs |
| shadcn/ui | https://ui.shadcn.com |
| TanStack Query | https://tanstack.com/query |
| Vitest | https://vitest.dev |
| Playwright | https://playwright.dev |
| Loi 43-20 Maroc | https://www.dgssi.gov.ma |
| Loi 09-08 CNDP | https://www.cndp.ma |
| Normes BAM | https://www.bkam.ma |
| DNSSI guide | https://www.dgssi.gov.ma/fr/contenu/guide-implementation-politique-securite |

---

*Document vivant — mettre à jour à chaque sprint. Version 1.0 — Avril 2026*
*Programme Parapheur Digital — Banque Marocaine Cotée — Solo Developer + Claude*
# Workflow Claude Code + VS Code — Développement pas à pas
## Parapheur Digital : de la spec au code, session par session
**Guide pratique d'interaction — Solo Developer — Avril 2026**

---

## AVANT DE COMMENCER : Qu'est-ce que Claude Code ?

Claude Code est un **agent de développement** qui s'exécute dans votre terminal ou dans VS Code. Contrairement à ChatGPT ou Claude.ai (interface web), Claude Code :

- **Lit vos fichiers** directement (specs, code existant, tests)
- **Écrit et modifie** du code dans votre projet
- **Exécute des commandes** terminal (npm, git, prisma, vitest...)
- **Voit votre arborescence** et navigue dedans
- **Itère en autonomie** : il peut enchaîner lecture → écriture → test → correction

Vous n'avez pas besoin de copier-coller du code. Claude Code travaille *dans* votre projet.

---

## PARTIE 1 — INSTALLATION ET CONFIGURATION

### Étape 1.1 — Installer Claude Code

```bash
# Dans votre terminal
npm install -g @anthropic-ai/claude-code

# Vérifier l'installation
claude --version
```

### Étape 1.2 — Connecter votre compte Anthropic

```bash
claude login
# Un navigateur s'ouvre → connectez-vous avec votre compte Anthropic
# Le token est sauvegardé automatiquement
```

### Étape 1.3 — Ouvrir VS Code avec Claude Code

```bash
# Aller dans votre dossier de projet
mkdir parapheur-digital-app
cd parapheur-digital-app
git clone https://github.com/tkassi/parapheur-digital.git specs
# (les specs sont dans ./specs/)

# Ouvrir VS Code
code .

# Dans le terminal VS Code (Ctrl+` pour l'ouvrir), lancer Claude Code
claude
```

Vous verrez apparaître le prompt interactif :
```
Claude Code — Anthropic
Working directory: /Users/vous/parapheur-digital-app
> _
```

### Étape 1.4 — Configurer CLAUDE.md (mémoire du projet)

Claude Code lit automatiquement un fichier `CLAUDE.md` à la racine. C'est sa **mémoire permanente** du projet.

**Dans le terminal Claude Code, tapez :**
```
/init
```

Claude va générer un `CLAUDE.md` de base. Ensuite, demandez-lui de le compléter :

```
Lis le fichier specs/PHASE4-V2-architecture-integration-MAROC-SOLO.md 
et specs/GUIDE-DEVELOPPEMENT-PARAPHEUR-DIGITAL.md, puis mets à jour 
CLAUDE.md avec :
- Le nom du projet et son contexte (banque marocaine, Loi 43-20)
- La stack technique (Node.js 20 + Fastify + TypeScript + Prisma + React)
- La structure monorepo (apps/ + packages/)
- Les 5 microservices et leurs responsabilités
- Les conventions de code (pas d'any, Zod validation, commits conventionnels)
```

Claude va lire les specs et générer un CLAUDE.md complet. **Ce fichier est lu à chaque session** — vous n'aurez plus besoin de réexpliquer le contexte.

---

## PARTIE 2 — PATTERN D'UNE SESSION DE DÉVELOPPEMENT

Chaque session suit ce pattern en 5 temps :

```
1. CONTEXTE    → Donner la spec à Claude (une seule fois par composant)
2. PLAN        → Demander le plan avant le code
3. CODE        → Faire écrire le code fichier par fichier
4. TEST        → Faire écrire et lancer les tests
5. REVIEW      → Demander une review, corriger, itérer
```

---

## PARTIE 3 — SESSION 1 : INITIALISATION DU MONOREPO (2h)

### Ouvrir Claude Code et démarrer

```bash
cd /Users/vous/parapheur-digital-app
claude
```

### Interaction 1 — Créer la structure monorepo

**Vous tapez :**
```
Lis le fichier specs/GUIDE-DEVELOPPEMENT-PARAPHEUR-DIGITAL.md section 3 
(Initialisation du monorepo). 

Crée la structure complète :
- package.json racine avec pnpm workspaces
- pnpm-workspace.yaml
- turbo.json
- .prettierrc
- packages/shared-config/tsconfig.base.json
- packages/shared-config/eslint-config/index.js
- .gitignore approprié pour Node.js + TypeScript

Ne génère pas encore les apps/, seulement la structure de base.
```

Claude va **créer tous ces fichiers** dans votre dossier. Vous voyez les fichiers apparaître dans l'explorateur VS Code en temps réel.

### Interaction 2 — Docker Compose

```
Crée maintenant le docker-compose.yml pour l'environnement de développement 
avec PostgreSQL 16, Redis 7, et Maildev. 
Utilise les credentials du fichier .env.example dans les specs.
Crée aussi .env.example et .env.local (gitignored).
```

### Interaction 3 — Vérification

```
Lance docker-compose up -d et vérifie que les 3 services démarrent correctement.
Montre-moi le résultat de docker-compose ps.
```

Claude va **exécuter la commande** dans votre terminal et vous montrer le résultat.

### Interaction 4 — Installer les dépendances

```
Installe les dépendances racine avec pnpm install.
Vérifie qu'il n'y a pas d'erreurs.
```

**Résultat attendu de la session 1 :** structure monorepo fonctionnelle, Docker up, dépendances installées. ~2 heures.

---

## PARTIE 4 — SESSION 2 : BASE DE DONNÉES PRISMA (3h)

### Ouvrir une nouvelle session

```bash
claude
```

> **Note :** Claude Code relit CLAUDE.md automatiquement → il connaît déjà le contexte.

### Interaction 1 — Créer le package shared-db

```
Lis specs/GUIDE-DEVELOPPEMENT-PARAPHEUR-DIGITAL.md section 4 (Base de données).

Initialise le package packages/shared-db :
1. Crée le package.json avec les dépendances prisma et @prisma/client
2. Initialise Prisma (npx prisma init)
3. Crée le schéma Prisma complet tel que décrit dans la section 4.2 du guide
   avec tous les modèles : User, Delegation, Dossier, Document, 
   WorkflowInstance, ValidationTask, WorkflowTransition, AuditLog
   et tous les enums : AuthLevel, DossierType, DossierStatus, TaskStatus, etc.
```

### Interaction 2 — Lancer la migration

```
Lance la première migration Prisma :
cd packages/shared-db && npx prisma migrate dev --name init

Ensuite crée le fichier de migration SQL pour le full-text search 
et les règles append-only sur audit_logs (section 4.3 du guide).
```

### Interaction 3 — Vérifier le schéma

```
Ouvre Prisma Studio pour que je puisse vérifier le schéma visuellement :
npx prisma studio
```

Claude lance Prisma Studio sur http://localhost:5555. Vérifiez les tables dans votre navigateur.

### Interaction 4 — Créer le seed

```
Crée le fichier packages/shared-db/prisma/seed.ts avec les 4 utilisateurs 
de test (Khalid N6, Aïcha N4, Mohammed N2, Yassir AUDITEUR) 
tels que définis dans specs/PHASE1-V2-specifications-detaillees-MAROC-SOLO.md 
section personas.

Lance ensuite le seed : npx prisma db seed
```

### Interaction 5 — Générer les types

```
Génère le client Prisma : npx prisma generate
Vérifie que les types TypeScript sont bien générés dans node_modules/@prisma/client
```

**Résultat attendu de la session 2 :** base de données PostgreSQL avec schéma complet, 4 utilisateurs de test, types TypeScript générés.

---

## PARTIE 5 — SESSION 3 : API GATEWAY + AUTHENTIFICATION (4h)

### Interaction 1 — Plan avant le code

```
Lis specs/PHASE1-V2-specifications-detaillees-MAROC-SOLO.md section 
"User Story US-01 : Authentification AD + MFA OTP".

Lis aussi specs/GUIDE-DEVELOPPEMENT-PARAPHEUR-DIGITAL.md section 5.

Donne-moi le plan de développement de l'API Gateway :
- Quels fichiers tu vas créer ?
- Dans quel ordre ?
- Quelles dépendances npm ?
Ne code rien encore, juste le plan.
```

**Attendez et lisez le plan.** Si quelque chose ne convient pas, corrigez avant de continuer. C'est le moment d'ajuster.

### Interaction 2 — Initialiser l'app

```
Plan approuvé. Commence par :
1. Créer apps/api-gateway/package.json avec toutes les dépendances
2. Créer apps/api-gateway/tsconfig.json (hérite de shared-config)
3. Créer apps/api-gateway/src/config.ts avec Zod (valide les variables d'env)
4. Créer apps/api-gateway/src/app.ts (Fastify avec plugins CORS, Helmet, 
   Rate limit, JWT)

Seulement ces 4 fichiers pour l'instant.
```

> **Conseil :** demandez toujours **4-5 fichiers maximum** par interaction. Claude est plus précis et vous pouvez vérifier au fur et à mesure.

### Interaction 3 — Route de login LDAP

```
Crée maintenant la route d'authentification :
- apps/api-gateway/src/routes/auth/login.ts
  (POST /auth/login → bind LDAP → JWT avec level)
- apps/api-gateway/src/routes/auth/totp.ts  
  (POST /auth/totp/verify → valide le code TOTP 6 chiffres)
- apps/api-gateway/src/middleware/authenticate.ts
  (hook Fastify qui vérifie le JWT sur toutes les routes protégées)

Pour le LDAP, utilise les variables LDAP_URL, LDAP_BASE_DN du .env.local.
```

### Interaction 4 — Test de la route login

```
Crée le fichier de test apps/api-gateway/src/__tests__/auth.test.ts avec Vitest.

Teste :
1. Login avec credentials valides → reçoit JWT + requireTotp: true pour N4
2. Login avec mauvais password → reçoit 401
3. Login avec username trop court → reçoit 400 (validation Zod)
4. Vérification TOTP valide → reçoit JWT final (totpVerified: true)
5. Vérification TOTP invalide → reçoit 401

Mock le client LDAP (ne pas appeler un vrai AD en test).
Lance les tests : pnpm --filter api-gateway test
```

Claude va écrire les tests ET les lancer. Vous verrez le résultat directement :

```
✓ auth > login valide → JWT (45ms)
✓ auth > mauvais password → 401 (12ms)
✓ auth > username court → 400 (8ms)
✓ auth > TOTP valide → JWT final (23ms)
✓ auth > TOTP invalide → 401 (11ms)

5 tests passed
```

### Interaction 5 — Si un test échoue

```
Le test "TOTP valide → JWT final" échoue avec cette erreur :
[coller l'erreur]

Analyse et corrige.
```

Claude va lire l'erreur, identifier la cause, modifier le code, et relancer les tests.

### Interaction 6 — Point d'entrée et démarrage

```
Crée apps/api-gateway/src/index.ts (point d'entrée).
Lance l'API : pnpm --filter api-gateway dev
Teste avec curl que GET /health répond 200.
```

**Résultat attendu session 3 :** API Gateway démarre sur port 3000, auth LDAP + TOTP fonctionnels, 5 tests verts.

---

## PARTIE 6 — SESSION 4 : MS-1 SERVICE DOSSIER (4h)

### Interaction 1 — Lire la spec métier

```
Lis specs/PHASE2-V2-specifications-metier-MAROC-SOLO.md section 
"Types de dossiers et volumétrie" et "Règles de gestion Dossier".

Lis aussi specs/PHASE1-V2-specifications-detaillees-MAROC-SOLO.md 
section "Epics B — Dossiers (US-05 à US-12)".

Résume en 10 bullets ce que MS-1 doit faire, selon ces specs.
```

Cette étape force Claude à **synthétiser les specs** avant de coder. Vous validez la compréhension.

### Interaction 2 — Repository pattern

```
Crée apps/ms-dossier/src/repositories/DossierRepository.ts

Ce repository doit exposer :
- create(data: CreateDossierInput): Promise<Dossier>
- findById(id: string): Promise<Dossier | null>
- findAll(filters: DossierFilters, pagination: Pagination): Promise<PaginatedResult<Dossier>>
- update(id: string, data: UpdateDossierInput): Promise<Dossier>
- findByReference(ref: string): Promise<Dossier | null>

Utilise le client Prisma de @parapheur/db.
Toutes les méthodes sont typées strictement (pas d'any).
```

### Interaction 3 — Service métier

```
Crée apps/ms-dossier/src/services/DossierService.ts

Ce service utilise DossierRepository et implémente :
- createDossier() : génère la référence DOS-YYYY-NNNN, convertit la devise via 
  l'API BAM (cache Redis 1h), publie un event BullMQ 'dossier.created'
- submitDossier() : vérifie que le statut est BROUILLON, publie 'dossier.submitted'
- cancelDossier() : vérifie les permissions (seul le créateur peut annuler)
- duplicateDossier() : copie un dossier sans ses documents ni son workflow

Pour la conversion devise, utilise specs/PHASE2-V2 section "Multi-devise BAM".
```

### Interaction 4 — Routes Fastify

```
Crée les routes Fastify pour MS-1 :
- POST   /dossiers          → createDossier
- GET    /dossiers          → findAll (avec pagination + filtres)
- GET    /dossiers/:id      → findById
- PATCH  /dossiers/:id      → update (brouillon seulement)
- POST   /dossiers/:id/submit   → submitDossier
- POST   /dossiers/:id/cancel   → cancelDossier
- POST   /dossiers/:id/duplicate → duplicateDossier

Chaque route a un schéma Zod pour validation du body et des params.
Codes d'erreur : 400 (validation), 401 (non auth), 403 (interdit), 
404 (not found), 422 (business rule violation).
```

### Interaction 5 — Tests du service

```
Crée apps/ms-dossier/src/__tests__/DossierService.test.ts

Teste :
1. Création dossier MAD 800 000 → référence générée DOS-2026-0001
2. Création dossier EUR → conversion MAD (mock API BAM rate 10.85)
3. Soumission dossier BROUILLON → statut SOUMIS + event publié
4. Soumission dossier déjà SOUMIS → erreur 422
5. Annulation par le créateur → OK
6. Annulation par un autre user → erreur 403
7. Duplication → nouveau dossier BROUILLON sans workflow

Coverage cible : 90% sur DossierService.
Lance : pnpm --filter ms-dossier test --coverage
```

### Interaction 6 — Corriger jusqu'au vert

```
[Si des tests échouent, copier l'output ici]
Corrige jusqu'à ce que tous les tests passent.
```

**Résultat attendu session 4 :** MS-1 complet, 7 endpoints REST, 7 tests verts, coverage ≥90%.

---

## PARTIE 7 — SESSION 5 : MS-2 WORKFLOW XSTATE (la plus complexe — 2 jours)

> **Conseil :** Prévoyez 2 sessions distinctes pour ce microservice. C'est le cœur métier.

### Session 5a — Machine XState (jour 1)

#### Interaction 1 — Lire les règles DMN

```
Lis specs/PHASE2-V2-specifications-metier-MAROC-SOLO.md section complète 
"87 règles DMN" et la section "Hiérarchie d'autorisation N1-N6".

Ensuite lis specs/PHASE3-V2-conformite-securite-MAROC-SOLO.md section 
"Loi 43-20 — Niveaux de signature SES/SEA/SEQ".

Explique-moi avec tes propres mots :
1. Comment se calcule la chaîne de validation pour un crédit de 800 000 MAD ?
2. Quel type de signature est requis ?
3. Qui valide dans quel ordre ?
```

Vous validez que Claude a bien compris les règles AVANT qu'il les code.

#### Interaction 2 — Fonction computeValidationChain

```
Crée packages/shared-schemas/src/dmnRules.ts

Cette fonction computeValidationChain(montantMad: number, type: DossierType) 
retourne un tableau de ValidationStep[] selon les règles suivantes 
[détailler les règles clés ou lui demander de les extraire des specs].

Crée aussi les tests packages/shared-schemas/src/__tests__/dmnRules.test.ts
avec 15 cas de test couvrant toutes les frontières de montant :
- 50 000 MAD (non-engagement) → SES N5
- 80 000 MAD (achat) → SES N5  
- 100 001 MAD (crédit) → SEA N5
- 549 999 MAD → SEA N5
- 550 001 MAD → SEA N4
- 2 200 001 MAD → SEA N3 + N4
- 5 000 001 MAD → double signature
- 11 000 001 MAD → N2 requis
- 50 000 001 MAD → N1 requis
... etc.

Lance les tests et assure-toi que les 15 passent.
```

#### Interaction 3 — Machine XState de base

```
Crée apps/ms-workflow/src/machines/dossierWorkflow.machine.ts

Machine XState 5 avec :
- États : brouillon, soumis, attente_valideur, en_attente_info, approuve, rejete
- Events : SUBMIT, APPROVE, REJECT, REQUEST_INFO, RESPOND_INFO, CANCEL
- Context : dossierId, montantMad, type, validationChain, currentStep, decisions

La machine doit être sérialisable en JSON pour persistence PostgreSQL.
Utilise computeValidationChain pour initialiser le context.

Pas encore les guards ni les actions, juste la structure des états et transitions.
```

#### Interaction 4 — Ajouter les guards et actions

```
Ajoute maintenant les guards à la machine :
- validationComplete : currentStep >= validationChain.length
- isCurrentValideur : l'acteur est bien le valideur attendu à cette étape
- canEscalate : le délai SLA est dépassé de 50%+

Et les actions :
- assignToCurrentValideur : crée une ValidationTask en base
- recordApproval : enregistre la décision, incrémente currentStep
- calculateSlaDeadline : N5→4h, N4→8h, N3→1j, N2→2j, N1→3j (jours ouvrés Maroc)

Utilise le calendrier marocain (weekend sam-dim + jours fériés) pour les SLA.
```

### Session 5b — Service et tests (jour 2)

#### Interaction 5 — WorkflowService

```
Crée apps/ms-workflow/src/services/WorkflowService.ts

Ce service :
1. startWorkflow(dossierId) : instancie la machine XState, persiste le snapshot JSON
2. processEvent(instanceId, event, actorId) : recharge le snapshot, 
   envoie l'event, repersiste, publie les events BullMQ appropriés
3. getWorkflowState(dossierId) : retourne l'état courant + tâches en attente
4. checkSlaBreaches() : job planifié toutes les heures, escalade automatique

Le snapshot XState est stocké dans workflow_instances.machine_snapshot (JSONB).
```

#### Interaction 6 — Tests du workflow

```
Crée apps/ms-workflow/src/__tests__/WorkflowService.test.ts

Simule le scénario complet du parcours Khalid (spec PHASE1 section P1) :
1. Khalid crée dossier crédit 800 000 MAD → workflow démarré
2. Étape 1 : assigné au N4 (Aïcha) → tâche créée avec SLA 8h
3. Aïcha approuve → étape suivante ou fin ?
4. Dossier finalement APPROUVE → état final atteint

Aussi tester :
5. Aïcha rejette → dossier REJETE directement
6. Aïcha demande info → état EN_ATTENTE_INFO
7. Khalid répond → retour ATTENTE_VALIDEUR
8. SLA dépassé → escalade déclenchée

Coverage cible : 95% (ce service est critique).
```

**Résultat attendu sessions 5a+5b :** Machine XState fonctionnelle, 87 règles DMN testées, workflow complet simulé de bout en bout.

---

## PARTIE 8 — SESSION 6 : FRONTEND REACT (3 sessions)

### Session 6a — Setup + Login (2h)

#### Interaction 1 — Initialiser Vite + Tailwind

```
Initialise le frontend apps/web avec Vite + React + TypeScript.
Installe et configure :
- Tailwind CSS avec le plugin RTL (tailwindcss-rtl)
- shadcn/ui (style default, couleurs slate)
- i18next + react-i18next
- React Router DOM
- TanStack Query v5
- React Hook Form + Zod

Configure tailwind.config.ts avec les couleurs du projet :
- primary: #1A365D (bleu marine)
- accent: #DD6B20 (orange)  
- success: #38A169 (vert)
- danger: #E53E3E (rouge)
```

#### Interaction 2 — Page de login

```
Lis specs/PHASE1-V2 section "Wireframe Screen 1 — Login AD+OTP".

Crée apps/web/src/pages/Login/LoginPage.tsx :
- Formulaire username + password (React Hook Form + Zod)
- Validation : username min 3 chars, password min 8 chars
- Appel POST /api/v1/auth/login via TanStack Query mutation
- Si requireTotp: true → rediriger vers TotpPage
- Messages d'erreur en FR (i18n)
- Design : bleu marine #1A365D, logo banque centré, formulaire carte blanche

Crée aussi TotpPage.tsx avec :
- Saisie code 6 chiffres (InputOTP shadcn)
- Compte à rebours 5 minutes
- Bouton "Renvoyer le code"
```

#### Interaction 3 — Voir le résultat

```
Lance le frontend : pnpm --filter web dev
Ouvre http://localhost:5173 dans le navigateur.
Fais un screenshot de la page de login.
```

> **Note :** Claude Code peut prendre des screenshots si vous avez un navigateur ouvert. Sinon, ouvrez-le vous-même et décrivez ce que vous voyez.

### Session 6b — Dashboard + Liste dossiers (3h)

#### Interaction 1 — Dashboard

```
Lis specs/PHASE1-V2 section "Wireframe Screen 2 — Dashboard" et 
"User Story US-29 Dashboard personnel".

Crée DashboardPage.tsx avec :
- 4 cartes statistiques : À valider (rouge si >0), Mes brouillons, 
  En cours, Risque SLA (orange)
- Widget "Mes tâches urgentes" : 3 premiers dossiers à valider triés urgence
- Bouton "Nouveau dossier" (orange, proéminent)

Utilise TanStack Query pour fetcher GET /api/v1/dashboard/stats
(mock la réponse pour l'instant si le backend n'est pas encore connecté).
```

#### Interaction 2 — Liste des dossiers

```
Crée DossierListPage.tsx avec :
- Tableau avec colonnes : Référence, Type, Contrepartie, Montant MAD, 
  Statut (badge coloré), Créé le, Actions
- Barre de recherche full-text (debounce 300ms)
- Filtres : Type, Statut, Période
- Pagination (10 par page)
- Clic sur une ligne → DossierDetailPage

Utilise les types Zod de @parapheur/shared-schemas pour typer la réponse API.
```

### Session 6c — Wizard de création dossier (3h)

```
Lis specs/PHASE1-V2 section "US-05 Créer un dossier" et 
"Wireframe Screen 3 — Wizard création étape 2".

Crée DossierCreatePage.tsx — wizard 4 étapes avec barre de progression :

Étape 1 — Type et montant :
  - Select type dossier
  - Input montant avec select devise (MAD/EUR/USD)
  - Affichage automatique équivalent MAD (appel API BAM)
  - Lookup contrepartie (input + dropdown résultats en temps réel)

Étape 2 — Détails :
  - Textarea objet (bilingue optionnel)
  - Date besoin (date picker)
  - Toggle urgence
  - Toggle dérogation (avec justification obligatoire si coché)

Étape 3 — Documents :
  - Drag & drop upload (max 50MB, PDF/DOCX/XLSX/PNG)
  - Liste des documents requis selon type dossier (spec PHASE2 DMN R-DOC-*)
  - Indicateur : requis vs optionnel

Étape 4 — Récapitulatif :
  - Résumé de toutes les informations
  - Affichage de la chaîne de validation qui sera appliquée
  - Bouton "Soumettre" (appel POST /dossiers puis POST /dossiers/:id/submit)

Validation Zod à chaque étape. Le bouton "Suivant" est désactivé si l'étape est invalide.
```

---

## PARTIE 9 — PATTERN POUR LES SESSIONS SUIVANTES

Pour chaque composant restant (MS-3 Notifications, MS-4 Audit, MS-5/6 Adapters, 
page de validation, i18n arabe, tests e2e...), répétez toujours ce pattern :

### Template de prompt d'ouverture de session

```
Contexte (Claude relit CLAUDE.md automatiquement).

Aujourd'hui je travaille sur : [NOM DU COMPOSANT]

Spec de référence : lis [FICHIER SPEC] section [SECTION PRÉCISE]

État actuel du code :
- Ce qui est déjà fait : [liste]
- Ce qui manque : [liste]

Objectif de cette session :
- Livrable 1 : [fichier/fonctionnalité]
- Livrable 2 : [fichier/fonctionnalité]

Donne-moi le plan avant de commencer.
```

### Template pour demander un fichier précis

```
Crée [chemin/fichier.ts] qui :
1. [fonctionnalité 1]
2. [fonctionnalité 2]
3. [fonctionnalité 3]

Contraintes :
- Pas d'any TypeScript
- Gestion d'erreurs avec codes HTTP appropriés
- [contrainte métier spécifique]

Ensuite crée le test correspondant et lance-le.
```

### Template pour corriger une erreur

```
Cette commande [commande] produit cette erreur :

[coller l'erreur complète avec stacktrace]

Analyse la cause racine (pas juste le symptôme) et propose la correction.
Après correction, relance la commande pour vérifier.
```

### Template pour une code review

```
Fais une code review de [fichier.ts].
Critères : sécurité (OWASP), performance, lisibilité TypeScript, 
gestion d'erreurs, couverture des cas limites.
Note chaque problème : CRITIQUE / MAJEUR / MINEUR.
Propose les corrections pour les CRITIQUE et MAJEUR.
```

---

## PARTIE 10 — COMMANDES CLAUDE CODE UTILES

### Commandes slash disponibles dans Claude Code

| Commande | Usage |
|----------|-------|
| `/init` | Génère CLAUDE.md pour le projet |
| `/clear` | Efface la conversation (garde CLAUDE.md) |
| `/compact` | Compresse le contexte (utile pour longues sessions) |
| `/cost` | Affiche le coût de la session en cours |
| `/doctor` | Vérifie la configuration Claude Code |

### Référencer des fichiers dans votre prompt

```
# Lire un fichier spécifique
Lis @apps/ms-dossier/src/services/DossierService.ts et explique ce qu'il fait.

# Lire plusieurs fichiers
Lis @packages/shared-db/prisma/schema.prisma et @apps/ms-workflow/src/machines/
et explique comment le workflow persiste son état.

# Lire un dossier entier
Lis tous les fichiers dans @apps/api-gateway/src/routes/auth/
```

### Exécuter des commandes terminal

```
# Claude peut exécuter n'importe quelle commande
Lance : pnpm --filter ms-dossier test --coverage --reporter=verbose

# Il voit le résultat et peut l'analyser
Lance : pnpm --filter ms-dossier build && echo "Build OK" || echo "Build FAILED"

# Git
Lance : git diff HEAD~1 --stat
```

---

## PARTIE 11 — ORGANISATION DES 315 JH EN SESSIONS CLAUDE CODE

| Phase | Sessions | Durée | Livrables |
|-------|----------|-------|-----------|
| **P5 Setup** | 2 sessions | 1 sem | Monorepo + DB + Docker |
| **P6 Backend L1** | 6 sessions | 3 sem | MS-1 Dossier + MS-4 Audit |
| **P6 Backend L2** | 8 sessions | 4 sem | MS-2 Workflow XState (sessions longues) |
| **P7 Frontend L1** | 6 sessions | 3 sem | Auth + Dashboard + Liste + Création |
| **P7 Frontend L2** | 4 sessions | 2 sem | Validation queue + Signature modal |
| **P8 Intégrations** | 6 sessions | 3 sem | MS-3 Notifs + MS-5/6 Adapters |
| **P8 i18n + RTL** | 3 sessions | 1,5 sem | Traductions FR/AR + layout RTL |
| **P8 Mobile PWA** | 2 sessions | 1 sem | Vue mobile + offline |
| **P8 Admin** | 3 sessions | 1,5 sem | Dashboard RSSI + Audit logs + Config DMN |
| **P9 Tests e2e** | 4 sessions | 2 sem | Playwright : 5 parcours complets |
| **P9 Sécurité** | 3 sessions | 1,5 sem | OWASP ZAP + SAST + pentest basique |
| **P9 Conformité** | 2 sessions | 1 sem | CNDP + BAM audit trail |
| **P10 CI/CD** | 2 sessions | 1 sem | GitHub Actions + Docker prod |
| **P10 Monitoring** | 2 sessions | 1 sem | Prometheus + Grafana + alertes |
| **Total** | **~53 sessions** | **~27 semaines** | Application complète |

> **Rythme recommandé :** 2 sessions/jour (matin + après-midi), 5 jours/semaine.  
> Une "session" = 2 à 4 heures de travail avec Claude Code.

---

## PARTIE 12 — RÉSOLUTION DES PROBLÈMES COURANTS

### Problème : Claude génère du code qui ne compile pas

```
Ce code ne compile pas. Voici l'erreur TypeScript :
[erreur tsc]

Corrige le problème de types. N'utilise pas 'any' comme solution de facilité.
```

### Problème : Claude "oublie" le contexte après une longue session

```
# Option 1 : compresser le contexte
/compact

# Option 2 : rappeler le contexte clé
Pour rappel, le projet utilise :
- Fastify 4 + TypeScript (pas Express)
- Prisma 5 + PostgreSQL (pas de SQL raw)  
- Zod pour toutes les validations (pas de joi, pas de yup)
- Pas d'any TypeScript nulle part
```

### Problème : Les tests passent mais le comportement en dev est différent

```
Les tests passent mais quand je teste manuellement avec curl :
curl -X POST http://localhost:3001/dossiers -H "Content-Type: application/json" \
  -d '{"type":"ENGAGEMENT_CREDIT","montant":800000,"devise":"MAD",...}'

J'obtiens : [erreur]
Mais mon test mocke [ce composant]. Le problème vient probablement de [X].
Analyse et corrige.
```

### Problème : Performance lente sur une route

```
La route GET /dossiers est lente (>2s) quand il y a 1000 dossiers.
Analyse la requête Prisma générée (active le query logging : 
log: ['query'] dans PrismaClient).
Propose les index manquants et optimise la requête.
```

---

## RÉCAPITULATIF — LES 5 RÈGLES D'OR AVEC CLAUDE CODE

**1. Toujours lire la spec avant de coder**
```
Lis specs/[FICHIER] section [X] → résume → valide → code
```

**2. Plan avant code pour les composants complexes**
```
Donne-moi le plan → je valide → tu codes
```

**3. Petits incréments (4-5 fichiers max par interaction)**
```
Crée ces 4 fichiers → teste → itère → 4 fichiers suivants
```

**4. Tests systématiques après chaque composant**
```
Crée le composant → crée les tests → lance les tests → corrige → green
```

**5. Commits après chaque session réussie**
```
Lance : git add -p && git commit -m "feat(ms-dossier): add DossierService with tests"
```

---

*Guide Claude Code Workflow — Parapheur Digital — Avril 2026*  
*Solo Developer + Claude Code → Application professionnelle en production*
